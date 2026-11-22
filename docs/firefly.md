# Firefly III

Finanzas de la familia, con su importador de datos. Los adjuntos de los movimientos están en `/docker/firefly/upload`.

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/firefly/claves` y crea su usuario y su base, `firefly`.

Firefly cifra los adjuntos y algunas preferencias con su clave, `APP_KEY`, guardada también en `/docker/firefly/claves`. No puede cambiar: con otra clave, lo ya cifrado no se puede leer. Es la clave de ejemplo de Firefly, la que tenía la instalación anterior.

## Traer Firefly desde su stack anterior

El Firefly anterior guardaba sus datos en MariaDB. El stack los pasa solo al compartido si encuentra un volcado en `/docker/firefly/traspaso/firefly.sql`:

1. `firefly-mariadb` carga el volcado en una MariaDB que solo vive en memoria.
2. `firefly-esquema` crea las tablas en PostgreSQL con las migraciones de Firefly, como en una instalación nueva.
3. `firefly-traspaso` copia los datos de una a otra con pgloader, deja la marca `traspaso/hecho` y renombra el volcado a `firefly.sql.restaurado`. Con la marca, los despliegues siguientes no hacen nada.

Pasos:

1. **Parar Firefly** en Portainer (`firefly_iii_core` y `firefly_iii_importer`), dejando MariaDB en marcha.
2. **Volcar la base** por SSH, con el usuario y la contraseña del stack anterior:

   ```bash
   sudo mkdir -p /docker/firefly/traspaso
   sudo docker exec mariadb mariadb-dump -u<usuario> -p<contraseña> --single-transaction fireflyiii | sudo tee /docker/firefly/traspaso/firefly.sql > /dev/null
   ```

3. **Quitar el stack anterior** en Portainer, sin borrar volúmenes.
4. **Desplegar el stack** `firefly`. El log de `firefly-traspaso` termina con un resumen de las filas copiadas.
5. **Desplegar el stack de Traefik,** que ya no declara las rutas de Firefly y su importador entre las rutas sin migrar: ahora las publican las etiquetas del stack.
6. Entrar con la cuenta de siempre y comprobar los saldos de las cuentas, algunos movimientos y un adjunto.

Pasados unos días con todo funcionando, se borra la base `fireflyiii` de MariaDB, que se va con ella en la limpieza final, y `/docker/firefly/traspaso`.

## Importador

El importador no guarda ninguna clave: al entrar, pide el *Client ID* de un cliente OAuth de Firefly y Firefly pide permiso. El cliente se crea una vez, en Firefly, en **Opciones → Perfil → OAuth → Crear nuevo cliente**, con la dirección de vuelta que muestra el importador en su página de inicio y sin marcar *Confidencial*.

## Copias

Hay que copiar tres cosas a la vez: su base en el PostgreSQL compartido, la carpeta `/docker/firefly/upload` y la clave de `/docker/firefly/claves/app-key`, sin la que los adjuntos no se pueden leer.

[← Volver al índice](../README.md)
