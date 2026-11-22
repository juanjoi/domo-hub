# Catálogo de servicios

Servicios definidos en este repositorio. Cada servicio se añade como una fila más al migrarlo o darlo de alta.

| Servicio | Categoría | Propósito |
| --- | --- | --- |
| Home Assistant | Casa | Control de la casa ([puesta en marcha](homeassistant.md)) |
| Mealie | Casa | Recetas, menú semanal y lista de la compra; se publica como «recetas» ([puesta en marcha](mealie.md)) |
| Quereseres | Casa | Tareas pendientes de casa en tableros kanban |
| 3tero | Casa | Proyectos de impresión 3D: modelos, fotos, materiales, coste e importación desde MakerWorld ([puesta en marcha](3tero.md)) |
| Immich | Media | Fotos y vídeos de la familia ([puesta en marcha](immich.md)) |
| Jellyfin | Media | Películas y series |
| File Browser | Media | Acceso web a los ficheros compartidos |
| Traefik | Administración | Proxy inverso: recibe las peticiones web y las reparte según el nombre |
| domo-board | Administración | Panel para actualizar los stacks con un botón |
| domo-menu | Infraestructura | Menú común: botón en el borde de cada aplicación web que abre la lista de aplicaciones |
| Zigbee2MQTT | Administración | Puente entre los dispositivos Zigbee y MQTT |
| Mosquitto (MQTT) | Infraestructura | Bus de mensajes entre Zigbee2MQTT y Home Assistant |
| Asistente de voz | Infraestructura | Whisper (voz a texto) y Piper (texto a voz) para Assist ([puesta en marcha](asistente-voz.md)) |
| PostgreSQL | Infraestructura | Base de datos compartida, una base por aplicación ([puesta en marcha](postgres.md)) |
| Traductor | Infraestructura | Traducción automática en casa con LibreTranslate; lo usa 3tero ([puesta en marcha](traductor.md)) |

Categorías: **Casa**, **Media**, **Administración**, **Infraestructura**. Administración es lo que se gestiona a mano, como el proxy, el panel o Zigbee2MQTT; Infraestructura, los servicios de fondo de los que dependen los demás.

[← Volver al índice](../README.md)
