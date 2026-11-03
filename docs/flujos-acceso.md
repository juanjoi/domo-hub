# Flujos de acceso

El acceso funciona igual desde casa que desde fuera. La única diferencia es que, fuera de casa, Tailscale abre antes el túnel.

## Desde casa

1. El dispositivo pide un nombre interno (por ejemplo, el de un servicio familiar).
2. Pi-hole lo traduce a la dirección del servidor.
3. La petición llega a Traefik.
4. Traefik la reenvía al contenedor que corresponde según el nombre pedido.

## Desde fuera de casa

1. El dispositivo se conecta a Tailscale.
2. Las consultas DNS de los nombres internos llegan a Pi-hole a través del túnel.
3. Desde aquí el recorrido es el mismo que desde casa: primero Traefik y luego el contenedor.

> **Por confirmar:** se da por hecho que Tailscale usa Pi-hole como DNS (split DNS).

Ningún servicio está expuesto directamente a Internet: todo el acceso desde fuera pasa por Tailscale.

[← Volver al índice](../README.md)
