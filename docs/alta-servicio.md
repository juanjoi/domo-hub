# Alta de un servicio nuevo

Publicar un servicio nuevo obliga a tocar tres sitios, en este orden: este repositorio, el servidor y Pi-hole. El proxy no se toca: el propio stack declara cómo se publica.

1. **Repositorio**: añadir `stacks/<servicio>/docker-compose.yml`, con sus datos bajo `/docker/<servicio>` y un `.env.example` si necesita secretos, y subirlo a `main`. Si se publica por el proxy, conectarlo a la red `proxy` y añadirle las etiquetas de Traefik (ver [publicar un servicio](operacion.md#publicar-un-servicio)).
2. **Servidor**: crear el `.env` con los valores reales y desplegar el stack (ver [operación](operacion.md)).
3. **Pi-hole**: añadir el registro DNS local con el nombre interno del servicio, apuntando al servidor.
4. **Comprobar**: abrir el servicio por su nombre desde casa y también desde fuera, con Tailscale.
5. **Documentar**: añadirlo al [catálogo de servicios](catalogo-servicios.md).

[← Volver al índice](../README.md)
