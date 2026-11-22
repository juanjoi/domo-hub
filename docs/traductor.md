# Traductor

Traductor automático compartido, con LibreTranslate. No necesita cuentas ni claves y los textos no salen de casa. Hoy lo usa 3tero, para traducir al español los modelos que se importan.

El stack crea la red `traductor`. Solo se unen a ella las aplicaciones que traducen. No se publica por Traefik ni publica puertos: las aplicaciones lo llaman por su nombre, `http://traductor:5000`.

## Primer despliegue

1. Desplegar el stack antes que las aplicaciones que lo usan, porque crea su red.
2. La primera vez descarga los modelos de idiomas (inglés, español, alemán, francés, italiano y chino) en `/docker/traductor/modelos`: un par de GB y varios minutos. Hasta que termina no responde, y las aplicaciones avisan de que no pueden traducir.

## Uso

- Va con el procesador: una descripción larga tarda en torno a medio minuto, y la primera traducción tras arrancar, algo más.
- Para añadir un idioma, se añade su código a `LT_LOAD_ONLY` y se despliega el stack.
- Para que una aplicación nueva lo use, se une a la red `traductor`, declarada como externa.

## Copias

No hace falta copiar nada: los modelos se vuelven a descargar solos.

[← Volver al índice](../README.md)
