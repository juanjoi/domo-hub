# Home Assistant

Home Assistant se instala desde cero y recupera los dispositivos Zigbee a través de MQTT: Zigbee2MQTT conserva el emparejamiento y los anuncia a Home Assistant en cuanto este se conecta.

## Primer arranque

Recién instalado, Home Assistant no acepta peticiones a través de un proxy: por Traefik responde con un error 400. La configuración del servidor HTTP, incluido el proxy, se gestiona desde la interfaz y no desde `configuration.yaml`. Por eso la primera vez se entra directamente al contenedor con un túnel SSH:

1. **Desplegar el stack** con la carpeta de datos vacía.
2. **Buscar la IP del contenedor** en la red `proxy`:

   ```bash
   sudo docker inspect homeassistant --format '{{range $n, $c := .NetworkSettings.Networks}}{{$n}} {{$c.IPAddress}}{{println}}{{end}}'
   ```

3. **Abrir un túnel** desde el equipo, con `ssh -L 8123:<IP del contenedor>:8123 <servidor>`, y entrar en `http://localhost:8123`.
4. **Completar el asistente inicial**: usuario, zona y unidades.
5. **Configurar el proxy** en los ajustes de red de Home Assistant: activar el proxy inverso y confiar en la subred de la red `proxy`, que está fijada en el stack de Traefik. Home Assistant se reinicia y pide confirmar el cambio. Hay que confirmarlo, todavía por el túnel, antes de 5 minutos. Si no, lo descarta.
6. **Comprobar** que Home Assistant abre por su nombre a través de Traefik.

Si algún día cambia la subred de `proxy`, hay que cambiarla también aquí.

## Zigbee y MQTT

1. **Conectar MQTT:** *Ajustes → Dispositivos y servicios → Añadir integración → MQTT*, con el broker `mqtt` y su puerto por defecto, sin usuario ni contraseña.
2. **Comprobar los dispositivos Zigbee:** aparecen solos en la integración MQTT. Si no aparecen, revisar que Zigbee2MQTT tiene activada la integración con Home Assistant en su configuración.
3. **Ordenar:** asignar áreas y nombres. Los nombres de los dispositivos vienen de Zigbee2MQTT; si se cambian allí, se cambian también aquí.

Los dispositivos Wi-Fi, como ESPHome o los satélites de voz, no se descubren solos, porque Home Assistant no usa la red del host. Se añaden con su integración indicando su IP.

[← Volver al índice](../README.md)
