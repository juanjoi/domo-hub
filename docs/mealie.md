# Mealie

Recetas, menú semanal y lista de la compra. Se publica como «recetas».

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/mealie/claves` y crea su usuario y su base, `mealie`. Las imágenes de las recetas y las copias de Mealie están en `/docker/mealie/data`.

Al usuario `mealie` se le permite además desactivar las claves ajenas en su propia sesión, algo que de serie solo puede el superusuario: Mealie lo necesita para restaurar sus copias de seguridad.

## Traer Mealie desde su stack anterior

El Mealie anterior guardaba sus datos en SQLite, en `/docker/mealie/data/mealie.db`: tenía `DB_ENGINE=postgresql`, un valor que Mealie no reconoce, y sin avisar usaba SQLite. El stack trae los datos solo: `mealie-traspaso`, antes de arrancar Mealie, pasa las tablas de la SQLite al compartido con el código de copias de la propia Mealie y la renombra a `mealie.db.traspasada`. Sin SQLite, no hace nada. Si la base del compartido ya tiene datos, falla avisando en vez de escribir encima.

1. **Copia de seguridad**, por si hay que volver atrás: en el Mealie anterior, **Ajustes → Copias de seguridad**, crear una y descargarla.
2. **Quitar el stack anterior** en Portainer, sin borrar volúmenes.
3. **Desplegar el stack** `mealie`. El log de `mealie-traspaso` dice cuántas recetas, usuarios y listas ha pasado.
4. **Desplegar el stack de Traefik,** que ya no declara la ruta de Mealie entre las rutas sin migrar: ahora la publican las etiquetas del stack.
5. Entrar con las cuentas de siempre y comprobar las recetas con sus fotos y la lista de la compra.

Pasados unos días con todo funcionando, se borra `/docker/mealie/data/mealie.db.traspasada`. La base `mealie` del PostgreSQL antiguo estaba vacía y se va con él.

Para volver atrás: renombrar `mealie.db.traspasada` a `mealie.db` y volver a crear el stack anterior en Portainer.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/mealie/data`. Las copias de seguridad de Mealie, desde **Ajustes → Copias de seguridad**, también lo guardan todo y se restauran desde la misma pantalla.

[← Volver al índice](../README.md)
