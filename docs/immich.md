# Immich

Fotos y vídeos de la familia. Las fotos, las miniaturas y los vídeos convertidos están en `/docker/immich/library`; los modelos del reconocimiento de caras y la búsqueda, en `/docker/immich/model-cache`, y se descargan solos si faltan.

Usa el [PostgreSQL compartido](postgres.md), que lleva la imagen de PostgreSQL de Immich. El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/immich/claves` y crea su usuario y su base, `immich`. Immich no es superusuario, así que sus extensiones (VectorChord, pgvector y otras) las crea el contenedor de alta, que en cada despliegue también las actualiza si el compartido trae una versión nueva y, si cambia VectorChord, rehace sus índices.

Por no ser superusuario, las copias automáticas de la base que hace Immich fallan: se desactivan en **Administración → Ajustes → Copia de seguridad**. La base entra en las copias del servidor, como las demás.

## Traer Immich desde su stack anterior

El Immich anterior tenía su propio PostgreSQL, `immich_postgres`. El stack trae su base solo si encuentra un volcado en `/docker/immich/traspaso.dump`: `immich-traspaso` lo restaura antes de arrancar Immich, si la base del compartido aún no tiene tablas, y lo renombra a `traspaso.dump.restaurado`. Las fotos no se mueven: el stack usa la misma carpeta.

1. **Parar Immich** en Portainer: solo los contenedores `immich` e `immich_machine_learning`, dejando `immich_postgres` en marcha para volcar la base sin que cambie nada.
2. **Volcar la base** por SSH:

   ```bash
   sudo docker exec immich_postgres pg_dump -U immich -Fc immich | sudo tee /docker/immich/traspaso.dump > /dev/null
   ```

3. **Quitar el stack anterior** en Portainer, sin borrar volúmenes.
4. **Desplegar el stack** `immich`. El log de `immich-traspaso` dice cuántas fotos y vídeos ha pasado. La primera vez tarda algo más: el reconocimiento descarga sus modelos.
5. **Desplegar el stack de Traefik,** que ya no declara la ruta de Immich entre las rutas sin migrar: ahora la publican las etiquetas del stack.
6. Entrar con las cuentas de siempre, comprobar fotos, álbumes y caras, y desactivar las copias automáticas de la base, como se explica arriba. Las apps del móvil siguen funcionando: la dirección no cambia.

Pasados unos días con todo funcionando, se borran `/docker/immich/postgres` (la base anterior), `/docker/immich/traspaso.dump.restaurado` y el volumen `immich_model-cache` (los modelos anteriores).

## Actualizar

Antes de cambiar de versión, leer las notas de las versiones intermedias: Immich avisa de los cambios que piden tocar algo. Después de actualizar el PostgreSQL compartido, desplegar también este stack, para que el contenedor de alta actualice las extensiones.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/immich/library`.

[← Volver al índice](../README.md)
