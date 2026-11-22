# 3tero

Proyectos de impresión 3D de la familia: modelos, fotos, materiales, coste e importación desde MakerWorld. La imagen se publica desde su propio repositorio al etiquetar cada versión.

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/3tero/claves` y crea su usuario y su base, `trestero`.

## Primer despliegue

1. Desplegar antes el [PostgreSQL compartido](postgres.md). Sin él, el despliegue de 3tero falla avisando.
2. Desplegar el stack. No necesita `.env`. La primera vez da de alta la base en el compartido y crea la carpeta de ficheros en `/docker/3tero/data`. La aplicación crea sus tablas sola.
3. Abrir 3tero por su nombre y comprobar que carga.

## Traducción al español

Al importar un modelo, 3tero puede traducir el título, la descripción y las etiquetas al español con `3tero-traductor`, un LibreTranslate que va en el mismo stack. No necesita cuentas ni claves y los textos no salen de casa.

- En el primer despliegue descarga los modelos de idiomas (inglés, español, alemán, francés, italiano y chino) en `/docker/3tero/traductor`: un par de GB y varios minutos. Mientras tanto, el botón de traducir da un aviso y el resto de la aplicación funciona igual.
- Va con el procesador: una descripción larga tarda en torno a medio minuto, y la primera traducción tras arrancar, algo más.
- Para añadir un idioma, se añade su código a `LT_LOAD_ONLY` y se despliega el stack.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/3tero/data`, con las imágenes y los modelos. La aplicación también descarga una copia completa en un ZIP, desde **Ajustes → Copia de seguridad**, y con ella se puede restaurar todo.

[← Volver al índice](../README.md)
