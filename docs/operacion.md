# Operación

Este repositorio es la fuente de verdad: lo que está en `main` es lo que debe correr en el servidor. Los cambios se aplican a mano, stack a stack.

## Qué vive dónde

| Qué | Dónde |
| --- | --- |
| Definición de cada stack | `stacks/<servicio>/docker-compose.yml` en este repositorio |
| Secretos | `.env` junto al compose en el clon del servidor, fuera de Git, y en el gestor de contraseñas |
| Datos | `/opt/domo/<servicio>` en el servidor, cubierto por las copias de seguridad |

## Desplegar un cambio

1. Subir el cambio a `main`.
2. Aplicarlo en el servidor:
   - **Con el panel de actualización:** pulsar *Actualizar* en el stack.
   - **Mientras no exista el panel**, por SSH:

     ```bash
     cd <clon del repositorio> && git pull
     docker compose -f stacks/<servicio>/docker-compose.yml up -d
     ```

Todo cambio pasa por Git, salvo los secretos. Lo que se edite a mano en el servidor o en Portainer se pierde en el siguiente despliegue.

| Tarea | Cómo se hace |
| --- | --- |
| Servicio nuevo | [Alta de un servicio](alta-servicio.md) |
| Actualizar una imagen | Fusionar el PR de Renovate y desplegar el stack |
| Cambiar un secreto | Editar el `.env` en el servidor y desplegar el stack |
| Volver atrás | `git revert` y desplegar; si la versión nueva migró datos, restaurar también la copia |
| Dar de baja un servicio | Parar el stack, quitar su carpeta y decidir qué hacer con sus datos |

## Reglas de los stacks

- **Datos** en bind mounts con ruta absoluta bajo `/opt/domo/<servicio>`. Los volúmenes con nombre quedan para lo que se puede volver a descargar.
- **Versiones fijas** en las imágenes, no `latest` ni `stable`.
- **Secretos** en `.env`, con un `.env.example` versionado que lista los nombres.
- **Nombres de contenedor** claros y únicos, porque Nginx Proxy Manager los usa para encontrarlos.

## Red compartida

Los servicios publicados comparten la red `proxy`, para que Nginx Proxy Manager los alcance por nombre de contenedor. El resto se queda en la red propia de su stack, salvo las bases de datos compartidas.

- El stack de Nginx Proxy Manager crea la red, con `name: proxy`, y los demás la declaran como externa. Por eso se despliega antes que ellos.
- Los servicios detrás del proxy no publican puertos en el servidor.
- Home Assistant usa la red del host, así que no está en `proxy` y se llega a él a través del host.
- Los servicios de voz tampoco están en `proxy`: solo hablan con Home Assistant.

## Bases de datos compartidas

Un servidor de base de datos da servicio a varias aplicaciones, cada una con su propia base de datos y su propio usuario.

- El stack de la base de datos crea la red `db`, con `name: db`, y las aplicaciones que la usan la declaran como externa. Se despliega antes que ellas.
- Solo se unen a `db` las aplicaciones que usan esa base de datos. La base de datos no está en `proxy` ni publica puertos.
- Las copias hacen un volcado por base de datos, para poder restaurar una aplicación sin tocar las demás.
- Las aplicaciones con base de datos embebida (SQLite) no usan `db`.

[← Volver al índice](../README.md)
