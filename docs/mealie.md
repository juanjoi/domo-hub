# Mealie

Recetas, menú semanal y lista de la compra. Se publica como «recetas».

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/mealie/claves` y crea su usuario y su base, `mealie`. Las imágenes de las recetas y las copias de Mealie están en `/docker/mealie/data`.

Al usuario `mealie` se le permite además desactivar las claves ajenas en su propia sesión, algo que de serie solo puede el superusuario: Mealie lo necesita para restaurar sus copias de seguridad.

Venía de un stack anterior que, por un valor mal escrito en `DB_ENGINE`, guardaba los datos en SQLite. Se pasaron al compartido al migrarlo, y la SQLite quedó como `/docker/mealie/data/mealie.db.traspasada`: se borra cuando lleve unos días funcionando.

## Actualizar

Mealie actualiza su base sola al arrancar. Antes de desplegar una versión nueva:

1. Crear una copia en **Ajustes → Copias de seguridad**. Si algo sale mal, se vuelve a la versión anterior en el stack y se restaura esa copia.
2. Leer las notas de las versiones intermedias: los cambios que piden tocar la configuración vienen marcados como *breaking changes* o *things to know when upgrading*.

Desde la 3.26, Mealie solo hace peticiones a direcciones de internet: importar recetas, descargar imágenes, webhooks y acciones de recetas. Si alguno tuviera que llegar a un servicio de casa, hay que permitirlo con `HTTP_ALLOW_LIST`.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/mealie/data`. Las copias de seguridad de Mealie, desde **Ajustes → Copias de seguridad**, también lo guardan todo y se restauran desde la misma pantalla.

[← Volver al índice](../README.md)
