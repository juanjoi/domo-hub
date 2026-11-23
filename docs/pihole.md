# Pi-hole

DNS de casa: resuelve los nombres de los servicios y bloquea publicidad. Si se para, la casa se queda sin DNS, así que se despliega con cuidado.

Sus datos están en `/docker/pihole`: `pihole` (configuración, listas de bloqueo y estadísticas) y `dnsmasq.d` (configuración de dnsmasq). La web de administración no se publica por Traefik.

## Requisito: las reglas de los dominios internos

Pi-hole resuelve cualquier nombre de los dominios internos con la IP del servidor, y así un servicio nuevo no necesita registro DNS. Esas reglas llevan IPs y dominios de casa, así que **no están en Git**: se ponen a mano en el servidor, un fichero por dominio en `/docker/pihole/dnsmasq.d`, con una línea como esta:

```
address=/<dominio>/<IP>
```

Es lo único de este stack que hay que preparar a mano. Antes de arrancar Pi-hole, el stack comprueba que hay al menos una regla, y si no, se para con un aviso: sin ella no funcionaría ningún nombre de servicio. Cuando se monte un servidor desde cero, el instalador tendrá que pedir estos datos y escribir los ficheros.

Para cambiar una regla, se edita su fichero y se despliega el stack de nuevo.

## Desplegar

Desplegar el stack recrea el contenedor, y mientras arranca no hay DNS: hasta un minuto, aunque Docker lo dé por arrancado antes. Tampoco para el propio servidor, que resuelve a través de Tailscale, y Tailscale a través de Pi-hole: en ese minuto fallan `git fetch` y las descargas de imágenes. Se espera a que el contenedor salga como `healthy` antes de desplegar otra cosa. La imagen se descarga antes de parar el anterior, así que el corte se limita al arranque. Si Pi-hole no vuelve, se pone a mano un DNS público en el equipo desde el que se arregla.

La primera vez, al migrarlo desde Portainer, el contenedor anterior se llama igual y ocupa el mismo puerto. Para que el corte sea lo más corto posible, se hace por SSH en lugar de con el panel, que necesita DNS para descargar el repositorio:

```bash
cd /docker/domo-hub && sudo git fetch
sudo git restore --source=origin/main --worktree -- stacks/pihole/
sudo docker compose -f stacks/pihole/docker-compose.yml pull
sudo docker rm -f pihole
sudo docker compose -f stacks/pihole/docker-compose.yml up -d
```

Después se espera a que salga como `healthy` (`sudo docker ps --filter name=pihole`) y se borra el stack anterior en Portainer, sin borrar volúmenes.

## Actualizar

La versión es la misma que tenía la instalación anterior, la 5. La 6 cambia la configuración por completo: se guarda en `pihole.toml` y deja de leer `dnsmasq.d` si no se le indica. Se actualizará aparte, probándolo antes.

## Copias

Basta con copiar `/docker/pihole`. Las reglas de los dominios internos están ahí y no en Git, así que esta copia es la única que hay de ellas.

[← Volver al índice](../README.md)
