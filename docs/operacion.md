# Operación

Este repositorio es la fuente de verdad: lo que está en `main` es lo que debe correr en el servidor. Los cambios se aplican a mano, stack a stack.

## Qué vive dónde

| Qué | Dónde |
| --- | --- |
| Definición de cada stack | `stacks/<servicio>/docker-compose.yml` en este repositorio |
| Secretos | `.env` junto al compose en el clon del servidor, fuera de Git, y en el gestor de contraseñas |
| Datos | `/docker/<servicio>` en el servidor, cubierto por las copias de seguridad |

## Desplegar un cambio

1. Subir el cambio a `main`.
2. Aplicarlo en el servidor:
   - **Con el panel de actualización:** pulsar *Actualizar* en el stack.
   - **Mientras no exista el panel**, por SSH:

     ```bash
     cd <clon del repositorio> && git fetch
     git restore --source=origin/main --worktree -- stacks/<servicio>/
     docker compose -f stacks/<servicio>/docker-compose.yml up -d
     ```

   No se hace `git pull`: solo se actualiza la carpeta del stack que se despliega, igual que hace el panel. Con `git pull` cambiarían a la vez los ficheros de todos los stacks, y algunos, como las rutas de Traefik, se aplican al momento.

El panel se publica por Traefik como cualquier servicio, sin login. La primera vez se levanta a mano, y después se actualiza a sí mismo:

```bash
cd /docker/domo-hub
sudo docker compose -f stacks/domo-board/docker-compose.yml up -d
```

Si Traefik falla y no se llega al panel, se entra con un túnel SSH al puerto que el panel publica solo en el propio servidor, o se despliega por SSH con los comandos anteriores.

Todo cambio pasa por Git, salvo los secretos. Lo que se edite a mano en el servidor o en Portainer se pierde en el siguiente despliegue.

| Tarea | Cómo se hace |
| --- | --- |
| Servicio nuevo | [Alta de un servicio](alta-servicio.md) |
| Actualizar una imagen | Fusionar el PR de Renovate y desplegar el stack |
| Cambiar un secreto | Editar el `.env` en el servidor y desplegar el stack |
| Volver atrás | `git revert` y desplegar; si la versión nueva migró datos, restaurar también la copia |
| Dar de baja un servicio | Parar el stack, quitar su carpeta y decidir qué hacer con sus datos |

## Reglas de los stacks

- **Datos** en bind mounts con ruta absoluta bajo `/docker/<servicio>`. Los volúmenes con nombre quedan para lo que se puede volver a descargar.
- **Sin preparar nada a mano:** un stack se despliega desde el panel sin entrar al servidor. Si la imagen no corre como root y necesita que sus carpetas sean de su usuario, el stack incluye un contenedor de preparación que las crea y les da dueño antes de arrancar la aplicación (ver el de File Browser). Lo único que se prepara a mano son los `.env`.
- **Versiones fijas** en las imágenes, no `latest` ni `stable`.
- **Secretos** en `.env`, con un `.env.example` versionado que lista los nombres.
- **Nombres de contenedor** claros y únicos, porque Traefik los usa para encontrarlos.

## Red compartida

Los servicios publicados comparten la red `proxy`, para que Traefik los alcance por nombre de contenedor. El resto se queda en la red propia de su stack, salvo las bases de datos compartidas.

- El stack de Traefik crea la red, con `name: proxy`, y los demás la declaran como externa. Por eso se despliega antes que ellos.
- Los servicios detrás del proxy no publican puertos en el servidor.
- Home Assistant está en `proxy`, como cualquier servicio publicado, y en `domotica`.
- La red `domotica` la crea el stack de Mosquitto (MQTT). En ella están Zigbee2MQTT, Home Assistant y los servicios de voz, que no están en `proxy` porque solo hablan con Home Assistant.
- Durante la migración, Traefik también está en `services`, la red compartida anterior, para seguir llegando a los servicios que aún no se han migrado. Esos servicios se declaran en `stacks/traefik/dinamico/`. Cada servicio migrado pasa a `proxy` y a sus propias etiquetas.

## Publicar un servicio

Cada servicio publicado declara en su `docker-compose.yml` cómo lo publica Traefik, con etiquetas. Traefik las lee de Docker al arrancar el contenedor, sin reiniciarse.

```yaml
    labels:
      - traefik.enable=true
      - traefik.http.routers.<servicio>.rule=HostRegexp(`^<servicio>\.`)
      - traefik.http.services.<servicio>.loadbalancer.server.port=<puerto interno>
    networks:
      - proxy
```

- **Sin `traefik.enable=true`** no se publica nada: Traefik ignora el resto de contenedores.
- **La regla mira solo el principio del nombre**, así el dominio no aparece en este repositorio. Pi-hole envía al servidor cualquier nombre del dominio interno, así que un servicio nuevo no necesita registro DNS.
- **El puerto** es el interno del contenedor, siempre explícito.
- **Sin elegir entrada:** el servicio queda en todas las de Traefik, así que al activar HTTPS las etiquetas no cambian.
- **El panel de Traefik**, de solo lectura, se publica igual con el nombre `traefik` y muestra qué servicios ha encontrado.

## Bases de datos compartidas

Un servidor de base de datos da servicio a varias aplicaciones, cada una con su propia base de datos y su propio usuario.

- El stack de la base de datos crea la red `db`, con `name: db`, y las aplicaciones que la usan la declaran como externa. Se despliega antes que ellas.
- Solo se unen a `db` las aplicaciones que usan esa base de datos. La base de datos no está en `proxy` ni publica puertos.
- Las copias hacen un volcado por base de datos, para poder restaurar una aplicación sin tocar las demás.
- Las aplicaciones con base de datos embebida (SQLite) no usan `db`.

[← Volver al índice](../README.md)
