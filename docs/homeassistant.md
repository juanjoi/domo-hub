# Home Assistant

Home Assistant se instala desde cero y recupera los dispositivos Zigbee a través de MQTT: Zigbee2MQTT conserva el emparejamiento y los anuncia a Home Assistant en cuanto este se conecta.

## Primer arranque

Recién instalado, Home Assistant no acepta peticiones a través de un proxy: por Traefik respondería con un error 400. Si la carpeta de datos está vacía, el contenedor `homeassistant-config` crea la configuración inicial antes de arrancar Home Assistant. Es la configuración por defecto más el bloque `http`, que confía en la subred de la red `proxy`, fijada en el stack de Traefik. Si la configuración ya existe, no la toca.

Home Assistant no aplica ese bloque de forma definitiva. Lo pasa una sola vez a sus ajustes como cambio pendiente, y desde entonces se gestiona desde la interfaz:

1. **Desplegar el stack** con la carpeta de datos vacía y abrir Home Assistant por su nombre, a través de Traefik.
2. **Confirmar la configuración del proxy** cuando Home Assistant lo pida, antes de 5 minutos desde que arranca. Si no se confirma a tiempo, vuelve a la configuración sin proxy y responde con un error 400. Como el bloque del YAML ya no se vuelve a leer, entonces hay que empezar de cero: bajar el stack, borrar la carpeta de datos y desplegarlo de nuevo.
3. **Completar el asistente inicial**: usuario, zona y unidades.

Si algún día cambia la subred de `proxy`, hay que cambiarla también en los ajustes de red de Home Assistant.

## Zigbee y MQTT

1. **Conectar MQTT:** *Ajustes → Dispositivos y servicios → Añadir integración → MQTT*, con el broker `mqtt` y su puerto por defecto, sin usuario ni contraseña.
2. **Comprobar los dispositivos Zigbee:** aparecen solos en la integración MQTT. Si no aparecen, revisar que Zigbee2MQTT tiene activada la integración con Home Assistant en su configuración.
3. **Ordenar:** asignar áreas y nombres. Los nombres de los dispositivos vienen de Zigbee2MQTT; si se cambian allí, se cambian también aquí.

Los dispositivos Wi-Fi, como ESPHome o los satélites de voz, no se descubren solos, porque Home Assistant no usa la red del host. Se añaden con su integración indicando su IP.

[← Volver al índice](../README.md)
