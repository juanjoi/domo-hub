# Immich

Fotos y vídeos de la familia. Las fotos, las miniaturas y los vídeos convertidos están en `/docker/immich/library`; los modelos del reconocimiento de caras y la búsqueda, en `/docker/immich/model-cache`, y se descargan solos si faltan.

Usa el [PostgreSQL compartido](postgres.md), que lleva la imagen de PostgreSQL de Immich. El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/immich/claves` y crea su usuario y su base, `immich`. Immich no es superusuario, así que sus extensiones (VectorChord, pgvector y otras) las crea el contenedor de alta, que en cada despliegue también las actualiza si el compartido trae una versión nueva y, si cambia VectorChord, rehace sus índices.

Por no ser superusuario, las copias automáticas de la base que hace Immich fallan: se desactivan en **Administración → Ajustes → Copia de seguridad**. La base entra en las copias del servidor, como las demás.

Venía de un stack con su propio PostgreSQL: su base se pasó al compartido con un volcado al migrarlo. Cuando lleve unos días funcionando, se borran `/docker/immich/postgres` (la base anterior), `/docker/immich/traspaso.dump.restaurado` y el volumen `immich_model-cache` (los modelos anteriores).

## Actualizar

Antes de cambiar de versión, leer las notas de las versiones intermedias: Immich avisa de los cambios que piden tocar algo. Las apps del móvil se actualizan a la par que el servidor.

Tras migrar la base, Immich la limpia y avisa de que no puede limpiar algunas tablas internas de PostgreSQL (`permission denied to vacuum`): no son suyas, y es normal sin ser superusuario. Después de actualizar el PostgreSQL compartido, desplegar también este stack, para que el contenedor de alta actualice las extensiones.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/immich/library`.

[← Volver al índice](../README.md)
