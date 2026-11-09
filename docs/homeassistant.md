# Home Assistant

Home Assistant se instala desde cero y recupera los dispositivos Zigbee a través de MQTT: Zigbee2MQTT conserva el emparejamiento y los anuncia a Home Assistant en cuanto este se conecta.

## Primer arranque

Si la carpeta de datos está vacía, el contenedor `homeassistant-config` crea la configuración inicial antes de arrancar Home Assistant. Es la configuración por defecto más el bloque `http`, que hace que Home Assistant acepte las peticiones que le llegan a través de Traefik. Sin él, responde con un error 400. Si la configuración ya existe, no la toca.

Las peticiones solo se aceptan desde la red `proxy`, cuya subred está fijada en el stack de Traefik. Si algún día se cambia, hay que cambiarla también en `trusted_proxies`, dentro de la configuración de Home Assistant.

## Puesta en marcha

1. **Desplegar el stack** y abrir Home Assistant por su nombre. Completar el asistente inicial: usuario, zona y unidades.
2. **Conectar MQTT:** *Ajustes → Dispositivos y servicios → Añadir integración → MQTT*, con el broker `mqtt` y su puerto por defecto, sin usuario ni contraseña.
3. **Comprobar los dispositivos Zigbee:** aparecen solos en la integración MQTT. Si no aparecen, revisar que Zigbee2MQTT tiene activada la integración con Home Assistant en su configuración.
4. **Ordenar:** asignar áreas y nombres. Los nombres de los dispositivos vienen de Zigbee2MQTT; si se cambian allí, se cambian también aquí.

Los dispositivos Wi-Fi, como ESPHome o los satélites de voz, no se descubren solos, porque Home Assistant no usa la red del host. Se añaden con su integración indicando su IP.

[← Volver al índice](../README.md)
