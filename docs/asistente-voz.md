# Asistente de voz

El asistente de voz es **Assist**, el de Home Assistant, y todo ocurre en el servidor: ni el audio ni el texto salen de casa.

| Pieza | Qué hace | Dónde corre |
| --- | --- | --- |
| Satélite o app del móvil | Capta la voz y reproduce la respuesta | En cada habitación o en el móvil |
| Whisper | Pasa la voz a texto | Stack `asistente-voz` |
| Assist | Entiende el texto y actúa o responde | Home Assistant |
| Piper | Convierte la respuesta en voz | Stack `asistente-voz` |

Whisper y Piper hablan con Home Assistant por el protocolo Wyoming, a través de la red `voice`, que crea su stack. No publican puertos ni están en `proxy`. Los modelos se descargan la primera vez y se guardan en volúmenes de Docker, fuera de las copias, porque se pueden volver a descargar.

## Puesta en marcha

1. **Desplegar `asistente-voz`** y después actualizar `homeassistant`, que se une a la red `voice`. La primera vez Whisper y Piper tardan un rato en descargar los modelos: hay que esperar a que su log diga que están listos.
2. **Añadir Whisper y Piper:** en *Ajustes → Dispositivos y servicios → Añadir integración → Wyoming Protocol*, una vez para cada uno, con el nombre del contenedor como host (`wyoming-whisper`, `wyoming-piper`) y su puerto por defecto (10300 Whisper, 10200 Piper).
3. **Crear el asistente:** en *Ajustes → Asistentes de voz*, uno en español, con Whisper como voz a texto y Piper como texto a voz. Marcarlo como preferido.
4. **Probar por texto** desde el icono de Assist, escribiendo la pregunta. Es lo mismo que hará la voz, sin audio.
5. **Probar por voz** desde la app de Home Assistant en el móvil. El navegador no deja usar el micrófono sin HTTPS, así que desde la web solo se prueba por texto.
6. **Satélite:** se añade con la integración ESPHome indicando su IP, porque Home Assistant no lo descubre solo.

## Qué entiende

Con sus frases de serie, Assist solo ve las entidades **expuestas** (*Ajustes → Asistentes de voz → Exponer*) y las encuentra por su nombre, sus alias y su área. En la 2026.11 el nombre visible se compone del dispositivo, la entidad y el área, así que lo más fiable es darles **alias** que se digan con naturalidad, con los números en letra, porque así los escribe Whisper.

El asistente se crea con el idioma de la instalación: si sale en inglés, se cambia en *Ajustes → Asistentes de voz*, en el propio asistente.

### Ejemplo: ¿está abierta la ventana?

Con las frases de serie en español, un sensor de apertura (`binary_sensor`) solo se puede preguntar de una forma: verbo, nombre y estado, en ese orden. La respuesta es sí o no.

- «¿está la ventana abierta?», con «ventana» como nombre o alias de una entidad expuesta.
- «¿está la ventana cerrada en el salón?», con área.

No hay frases de serie para «¿está abierta la ventana?» (estado antes del nombre), «¿cómo está la ventana?» (solo para sensores con valor, como la temperatura) ni «¿hay alguna ventana abierta?» (solo para ventanas motorizadas, `cover`). Para eso hacen falta frases propias.

1. **Preparar el sensor.** Un sensor de apertura de Zigbee2MQTT llega como `binary_sensor` de tipo puerta. En los ajustes de la entidad, cambiar *Mostrar como* a **Ventana**, asignarle el área y, para las frases de serie, exponerlo y darle un alias. Para las frases propias no hace falta exponerlo ni darle alias.
2. **Añadir las frases** en `custom_sentences/es/` y las respuestas en `configuration.yaml`, con `intent_script`, dentro de la configuración de Home Assistant. Ninguno de los dos está en este repositorio: forman parte de los datos de Home Assistant. Las frases propias tienen prioridad sobre las de serie.
3. **Reiniciar Home Assistant** y probar en *Herramientas para desarrolladores → Assist*, que muestra qué intención ha reconocido, con qué datos y si la frase es de serie (`builtin`) o propia (`custom`).

`custom_sentences/es/ventanas.yaml`:

```yaml
language: "es"
intents:
  EstadoVentana:
    data:
      - sentences:
          - "(cómo|como) (está|esta) la ventana {ventana}"
          - "(está|esta) (abierta|cerrada) la ventana {ventana}"
          - "(está|esta) la ventana {ventana} (abierta|cerrada)"
  VentanasAbiertas:
    data:
      - sentences:
          - "hay (alguna|algunas) ventana[s] abierta[s]"
          - "(qué|que) ventanas (están|estan) abiertas"
lists:
  ventana:
    values:
      - in: "(del salón|del salon)"
        out: binary_sensor.<sensor de la ventana>
```

`configuration.yaml`:

```yaml
intent_script:
  EstadoVentana:
    speech:
      text: "La ventana está {{ 'abierta' if is_state(ventana, 'on') else 'cerrada' }}."
  VentanasAbiertas:
    speech:
      text: >
        {% set abiertas = states.binary_sensor
             | selectattr('attributes.device_class', 'eq', 'window')
             | selectattr('state', 'eq', 'on')
             | map(attribute='name') | list %}
        {% if abiertas %}Están abiertas: {{ abiertas | join(', ') }}.{% else %}No hay ninguna ventana abierta.{% endif %}
```

Cada ventana nueva es una línea más en `lists`, con el identificador de su entidad. *¿Hay alguna ventana abierta?* las incluye solas si tienen *Mostrar como: Ventana*.

## Aplicaciones propias

Una aplicación propia se integra igual que la ventana: frases propias asociadas a una acción, normalmente una llamada REST a la aplicación con `rest_command`, cuya respuesta se lee en voz alta.

[← Volver al índice](../README.md)
