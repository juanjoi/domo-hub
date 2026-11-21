# 3tero

Proyectos de impresión 3D de la familia: modelos, fotos, materiales, coste e importación desde MakerWorld. La imagen se publica desde su propio repositorio al etiquetar cada versión.

Necesita PostgreSQL. Mientras no esté desplegado el [PostgreSQL compartido](postgres.md), el stack trae su propia base de datos, `3tero-db`, en una red interna solo para la aplicación. Cuando esté el compartido, se pasa la base a él y se quita `3tero-db`.

## Primer despliegue

1. En el servidor, crear `stacks/3tero/.env` a partir de `.env.example`, con una contraseña larga solo de letras y números, porque va dentro de la URL de conexión. Guardarla también en el gestor de contraseñas.
2. Desplegar el stack. La primera vez crea la base en `/docker/3tero/postgres` y la carpeta de ficheros en `/docker/3tero/data`. La aplicación espera a que la base esté lista y crea sus tablas sola.
3. Abrir 3tero por su nombre y comprobar que carga.

## Copias

Hay que copiar dos cosas a la vez: la base de datos y la carpeta `/docker/3tero/data`, con las imágenes y los modelos. La aplicación también descarga una copia completa en un ZIP, desde **Ajustes → Copia de seguridad**, y con ella se puede restaurar todo.

## Pasar al PostgreSQL compartido

Con el compartido ya desplegado:

1. Descargar una copia desde la aplicación, por si hay que volver atrás.
2. Dar de alta el usuario y la base `trestero` en el compartido, como cualquier [aplicación nueva](postgres.md#dar-de-alta-una-aplicación).
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

5. Cambiar el stack en este repositorio:
   - Quitar el servicio `3tero-db` y la red `interna`.
   - Unir la aplicación a la red `db`, declarada como externa, en lugar de `interna`.
   - En `NUXT_DATABASE_URL`, cambiar el servidor `3tero-db` por `postgres`.
   - En el `.env`, poner la contraseña del usuario creado en el compartido.
6. Desplegar el stack y comprobar que los proyectos siguen ahí.
7. Pasados unos días con todo funcionando, borrar `/docker/3tero/postgres`.

[← Volver al índice](../README.md)
