# domo-hub

Stacks de Docker Compose del servidor doméstico (mini PC Intel N97, 16 GB RAM, sin GPU dedicada) y documentación de cómo usarlos: Pi-hole, Tailscale, Nginx Proxy Manager, Portainer, Home Assistant y servicios familiares. Repositorio público: nada de secretos, IPs, dominios ni datos de la familia. Si algo privado tiene que versionarse, se para y se revisa la decisión antes de subirlo.

## Convenciones

- Todo en español.
- `docs/`: documentación de uso, de alto nivel. Sin IPs, puertos ni credenciales salvo que se pida expresamente. El porqué de las decisiones no va aquí, sino en el repositorio de análisis.
- `stacks/<servicio>/docker-compose.yml`: un stack por servicio. Los secretos van en un `.env` junto al compose en el servidor, nunca en Git; se versiona un `.env.example` con los nombres.
- Datos en `/opt/domo/<servicio>` con ruta absoluta, imágenes con versión fija, red `proxy` solo para lo que se publica por Nginx Proxy Manager y red `db` solo para quien usa una base de datos compartida. Detalles en `docs/operacion.md`.
- Cada servicio nuevo se añade a `docs/catalogo-servicios.md`.
