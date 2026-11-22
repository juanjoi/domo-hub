# 3tero

Proyectos de impresión 3D de la familia: modelos, fotos, materiales, coste e importación desde MakerWorld. La imagen se publica desde su propio repositorio al etiquetar cada versión.

Necesita PostgreSQL. Mientras no esté desplegado el [PostgreSQL compartido](postgres.md), el stack trae su propia base de datos, `3tero-db`, en una red interna solo para la aplicación. No tiene contraseña: no publica puertos, no tiene salida a internet y solo la aplicación llega a ella. Cuando esté el compartido, se pasa la base a él y se quita `3tero-db`. Allí sí tendrá contraseña, pero la genera el propio stack: tampoco hará falta `.env`.

## Primer despliegue

1. Desplegar el stack. No necesita `.env`. La primera vez crea la base en `/docker/3tero/postgres` y la carpeta de ficheros en `/docker/3tero/data`. La aplicación espera a que la base esté lista y crea sus tablas sola.
2. Abrir 3tero por su nombre y comprobar que carga.

## Copias

Hay que copiar dos cosas a la vez: la base de datos y la carpeta `/docker/3tero/data`, con las imágenes y los modelos. La aplicación también descarga una copia completa en un ZIP, desde **Ajustes → Copia de seguridad**, y con ella se puede restaurar todo.

## Pasar al PostgreSQL compartido

Con el compartido ya desplegado, en dos cambios del stack. Ninguno necesita `.env`.

1. Descargar una copia desde la aplicación, por si hay que volver atrás.
2. **Primer cambio: dar de alta la base** en el compartido, sin mover todavía la aplicación. Añadir al stack el contenedor de alta, unido a la red `db` (declarada como externa), y declarar la red:

   ```yaml
   3tero-db-alta:
     # Misma imagen que el compartido, ya descargada, por psql.
     image: ghcr.io/immich-app/postgres:18-vectorchord1.1.1-pgvector0.8.7
     container_name: 3tero-db-alta
     entrypoint:
       - sh
       - -c
       - |
         set -e
         if [ ! -s /claves-postgres/postgres ]; then
           echo "Falta el PostgreSQL compartido: hay que desplegar antes su stack" >&2
           exit 1
         fi
         if [ ! -s /claves/postgres ]; then
           head -c 24 /dev/urandom | od -An -tx1 | tr -d ' \n' > /claves/postgres.nueva
           mv /claves/postgres.nueva /claves/postgres
         fi
         chown -R 1000:1000 /claves
         chmod 700 /claves
         chmod 400 /claves/postgres
         export PGHOST=postgres PGUSER=postgres PGPASSWORD="$$(cat /claves-postgres/postgres)"
         for i in $$(seq 30); do pg_isready -q && break; sleep 2; done
         psql -q -v ON_ERROR_STOP=1 -v usuario=trestero -v clave="$$(cat /claves/postgres)" <<'SQL'
         SELECT format('CREATE ROLE %I LOGIN', :'usuario')
           WHERE NOT EXISTS (SELECT FROM pg_roles WHERE rolname = :'usuario') \gexec
         ALTER ROLE :"usuario" PASSWORD :'clave';
         SELECT format('CREATE DATABASE %I OWNER %I', :'usuario', :'usuario')
           WHERE NOT EXISTS (SELECT FROM pg_database WHERE datname = :'usuario') \gexec
         REVOKE ALL ON DATABASE :"usuario" FROM PUBLIC;
         SQL
     volumes:
       - /docker/postgres/claves:/claves-postgres:ro
       - /docker/3tero/claves:/claves
     networks:
       - db
   ```

   Desplegar el stack. La aplicación sigue en la base temporal, y en el compartido queda la base `trestero` vacía.

3. Parar la aplicación, dejando `3tero-db` en marcha, para que no cambie nada a mitad:

   ```bash
   sudo docker stop 3tero
   ```

4. Volcar la base temporal y restaurarla en el compartido:

   ```bash
   sudo docker exec 3tero-db pg_dump -U trestero -Fc trestero \
     | sudo docker exec -i postgres pg_restore -U postgres -d trestero --no-owner --role=trestero
   ```

   Comprobar que no da errores y que las tablas principales, como `projects`, tienen el mismo número de filas en las dos bases.

5. **Segundo cambio: mover la aplicación.**
   - Quitar el servicio `3tero-db` y la red `interna`.
   - En la aplicación: depender de `3tero-db-alta` (que termine bien) en lugar de `3tero-db`, cambiar la red `interna` por `db`, montar `/docker/3tero/claves:/claves:ro`, quitar `NUXT_DATABASE_URL` de `environment` y arrancar leyendo la contraseña:

     ```yaml
     entrypoint:
       - sh
       - -c
       - |
         export NUXT_DATABASE_URL="postgres://trestero:$$(cat /claves/postgres)@postgres:5432/trestero"
         exec docker-entrypoint.sh node .output/server/index.mjs
     ```

     Si una versión nueva de la imagen cambia su orden de arranque, hay que cambiarla aquí también.
6. Desplegar el stack y comprobar que los proyectos siguen ahí.
7. Pasados unos días con todo funcionando, borrar `/docker/3tero/postgres`.

[← Volver al índice](../README.md)
