# Alta de un servicio nuevo

Publicar un servicio nuevo solo obliga a tocar dos sitios: este repositorio y el servidor. El proxy no se toca, porque el propio stack declara cómo se publica, y Pi-hole tampoco, porque envía al servidor cualquier nombre del dominio interno.

1. **Repositorio**: añadir `stacks/<servicio>/docker-compose.yml`, con sus datos bajo `/docker/<servicio>` y un `.env.example` si necesita secretos, y subirlo a `main`. Si se publica por el proxy, conectarlo a la red `proxy` y añadirle las etiquetas de Traefik (ver [publicar un servicio](operacion.md#publicar-un-servicio)).
2. **Servidor**: crear el `.env` con los valores reales y desplegar el stack (ver [operación](operacion.md)).
3. **Comprobar**: abrir el servicio por su nombre desde casa y también desde fuera, con Tailscale.
4. **Documentar**: añadirlo al [catálogo de servicios](catalogo-servicios.md).

[← Volver al índice](../README.md)
