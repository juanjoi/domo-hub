# Firefly III

Finanzas de la familia, con su importador de datos. Los adjuntos de los movimientos están en `/docker/firefly/upload`.

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/firefly/claves` y crea su usuario y su base, `firefly`.

Firefly cifra los adjuntos y algunas preferencias con su clave, `APP_KEY`, guardada también en `/docker/firefly/claves`. No puede cambiar: con otra clave, lo ya cifrado no se puede leer. Es la clave de ejemplo de Firefly, la que tenía la instalación anterior.

Venía de un stack con MariaDB: sus datos se pasaron al compartido al migrarlo, con pgloader, sobre las tablas creadas por las migraciones de Firefly. Cuando lleve unos días funcionando, se borran `/docker/firefly/traspaso` y la base `fireflyiii` de MariaDB, que se va con ella en la limpieza final.

## Actualizar

Firefly actualiza su base solo al arrancar. Antes de cambiar de versión, leer las notas de las versiones intermedias.

Desde la 6.7, las claves de API personales necesitan un cliente propio de Passport, que la instalación anterior no tenía. Si hace falta crear una clave en **Opciones → Perfil → OAuth**, se crea una vez por SSH:

```bash
sudo docker exec firefly php artisan passport:client --personal --provider=users --name="Firefly III Personal Access Client" --no-interaction
```

## Importador

El importador no guarda ninguna clave: al entrar, pide el *Client ID* de un cliente OAuth de Firefly y Firefly pide permiso. El cliente se crea una vez, en Firefly, en **Opciones → Perfil → OAuth → Crear nuevo cliente**, con la dirección de vuelta que muestra el importador en su página de inicio y sin marcar *Confidencial*.

## Copias

Hay que copiar tres cosas a la vez: su base en el PostgreSQL compartido, la carpeta `/docker/firefly/upload` y la clave de `/docker/firefly/claves/app-key`, sin la que los adjuntos no se pueden leer.

[← Volver al índice](../README.md)
