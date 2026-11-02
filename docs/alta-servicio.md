# Alta de un servicio nuevo

Publicar un servicio nuevo obliga a tocar cuatro sitios, en este orden: este repositorio, el servidor, Nginx Proxy Manager y Pi-hole.

1. **Repositorio**: añadir `stacks/<servicio>/docker-compose.yml`, con sus datos bajo `/opt/domo/<servicio>` y un `.env.example` si necesita secretos, y subirlo a `main`. Si se publica por el proxy, conectarlo a la red `proxy`.
2. **Servidor**: crear el `.env` con los valores reales y desplegar el stack (ver [operación](operacion.md)).
3. **Nginx Proxy Manager**: crear un *proxy host* con el nombre interno del servicio, apuntarlo al contenedor y asignarle un certificado.
4. **Pi-hole**: añadir el registro DNS local con ese nombre interno, apuntando al servidor.
5. **Comprobar**: abrir el servicio por su nombre desde casa y también desde fuera, con Tailscale.
6. **Documentar**: añadirlo al [catálogo de servicios](catalogo-servicios.md).

[← Volver al índice](../README.md)
