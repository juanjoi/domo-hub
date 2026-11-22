# PostgreSQL compartido

Un solo servidor PostgreSQL para las aplicaciones que lo necesitan, cada una con su propia base de datos y su propio usuario. Usa la imagen de PostgreSQL de Immich, que trae las extensiones vectoriales que Immich necesita; para el resto de aplicaciones es un PostgreSQL normal.

El stack crea la red `db`. Solo se unen a ella las aplicaciones que usan esta base de datos (ver [bases de datos compartidas](operacion.md#bases-de-datos-compartidas)).

Nada se prepara a mano: ni el servidor ni las aplicaciones necesitan `.env`. Las contraseñas las generan los propios stacks la primera vez y se quedan en el servidor, en una carpeta `claves` junto a los datos de cada uno.

## Primer despliegue

Desplegar el stack. La primera vez genera la contraseña del superusuario `postgres` en `/docker/postgres/claves` y crea el servidor vacío en `/docker/postgres/servidor`.

Esa contraseña no hace falta para administrar. Por SSH se entra sin ella, porque `psql` dentro del contenedor no la pide:

```bash
sudo docker exec -it postgres psql -U postgres
```

Solo la usan los contenedores que dan de alta las aplicaciones, que llegan por la red `db`. Desde la red siempre se pide contraseña.

## Dar de alta una aplicación

El stack de cada aplicación da de alta su base él solo, al desplegarse, con un contenedor de preparación (ver el de 3tero). Ese contenedor:

1. Genera la contraseña de la aplicación la primera vez, en `/docker/<aplicacion>/claves`, legible solo por el usuario con el que corre la aplicación.
2. Entra en el compartido como superusuario, con la contraseña de `/docker/postgres/claves`, que monta solo para leer.
3. Crea el usuario y la base de la aplicación si no existen, le pone su contraseña y cierra la base a los demás usuarios.

Se puede desplegar las veces que haga falta: si ya existen, no los toca. Si el compartido no está desplegado, falla avisando.

La aplicación no recibe la contraseña en una variable fija: su contenedor arranca con un `sh` que la lee del fichero, compone la dirección de la base y lanza la orden original de la imagen. Así sirve con imágenes de terceros sin cambiarlas; al actualizar una, hay que comprobar que su orden de arranque no ha cambiado.

Si la aplicación necesita extensiones, como Immich, las crea el superusuario o vienen en el volcado que se restaura, porque un usuario normal no puede crearlas.

## Traer la base de una aplicación

Para pasar una aplicación desde su base antigua, primero se despliega su stack con el contenedor de alta, para que exista la base vacía, y después se vuelca la antigua y se restaura aquí, con la aplicación parada para que no cambie nada a mitad:

```bash
sudo docker exec <contenedor antiguo> pg_dump -U <usuario> -Fc <base> \
  | sudo docker exec -i postgres pg_restore -U postgres -d <base> --no-owner --role=<usuario>
```

- **`--no-owner --role`** hace que todo lo restaurado sea del usuario de la aplicación, aunque en la base antigua el dueño se llamara de otra forma.
- **Versiones distintas:** el volcado se puede restaurar aquí aunque venga de una versión anterior de PostgreSQL.
- **Comprobar** antes de cambiar la aplicación: que `pg_restore` no da errores y que las tablas principales tienen el mismo número de filas en las dos bases.
- **Ensayo:** el mismo volcado se puede restaurar con la aplicación en marcha en una base de prueba, para comprobar que entra bien, sin tocarla. Después se borra la base de prueba (`DROP DATABASE`) y se repite con la aplicación parada.
- La base antigua no se borra hasta que la aplicación lleva unos días funcionando sobre el compartido.

## Actualizar

- **Versión menor** de PostgreSQL o de las extensiones: fusionar el PR de Renovate y desplegar, con un volcado de todas las bases antes. Los contenedores de alta usan la misma imagen, para no descargar otra: se actualizan a la vez.
- **Versión mayor** de PostgreSQL (por ejemplo, de la 18 a la 19): no basta con cambiar la imagen, porque los datos no son compatibles. Se vuelcan todas las bases, se levanta el servidor nuevo vacío y se restauran. La versión la marca Immich: se cambia cuando su imagen lo permite.

[← Volver al índice](../README.md)
