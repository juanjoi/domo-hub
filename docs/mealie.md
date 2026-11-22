# Mealie

Recetas, menú semanal y lista de la compra. Se publica como «recetas».

Usa el [PostgreSQL compartido](postgres.md). El stack se da de alta en él solo: genera la contraseña de la aplicación en `/docker/mealie/claves` y crea su usuario y su base, `mealie`. Las imágenes de las recetas y las copias de Mealie están en `/docker/mealie/data`.

## Traer Mealie desde su stack anterior

El Mealie anterior guardaba sus datos en SQLite, dentro de `/docker/mealie/data`: tenía `DB_ENGINE=postgresql`, un valor que Mealie no reconoce, y sin avisar usaba SQLite. Como cambia el motor, los datos no se pasan con un volcado de la base, sino con la copia de seguridad de Mealie, que vale para cualquier motor.

1. **Copia en el Mealie anterior:** en **Ajustes → Copias de seguridad**, crear una copia y descargarla también, por si hay que volver atrás. Se guarda en `/docker/mealie/data/backups`, que el nuevo también ve.
2. **Quitar el stack anterior** en Portainer, sin borrar volúmenes.
3. **Desplegar el stack** `mealie`. Arranca vacío sobre el compartido, con el usuario inicial de Mealie (`changeme@example.com`, contraseña `MyPassword`).
4. **Permitir la restauración.** Mealie desactiva las claves ajenas mientras restaura, y eso solo puede hacerlo el superusuario o a quien se le dé permiso. Se le da solo para este paso, por SSH:

   ```bash
   sudo docker exec postgres psql -U postgres -c "GRANT SET ON PARAMETER session_replication_role TO mealie"
   ```

5. **Restaurar:** entrar con el usuario inicial y, en **Ajustes → Copias de seguridad**, restaurar la copia del paso 1. Sustituye todo, usuarios incluidos: después se entra con las cuentas de siempre.
6. **Quitar el permiso:**

   ```bash
   sudo docker exec postgres psql -U postgres -c "REVOKE SET ON PARAMETER session_replication_role FROM mealie"
   ```

7. **Desplegar el stack de Traefik,** que ya no declara la ruta de Mealie entre las rutas sin migrar: ahora la publican las etiquetas del stack.
8. Comprobar que están las recetas con sus fotos y la lista de la compra.

Pasados unos días con todo funcionando, se borra `/docker/mealie/data/mealie.db`, la base SQLite anterior. La base `mealie` del PostgreSQL antiguo estaba vacía y se va con él.

## Copias

Hay que copiar dos cosas a la vez: su base en el PostgreSQL compartido y la carpeta `/docker/mealie/data`. Las copias de seguridad de Mealie, desde **Ajustes → Copias de seguridad**, también lo guardan todo y se pueden restaurar con los pasos 4 a 6.

[← Volver al índice](../README.md)
