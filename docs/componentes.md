# Componentes

La infraestructura se apoya en unas pocas piezas, y cada una tiene una sola responsabilidad.

| Componente | Función | Responsabilidad principal |
| --- | --- | --- |
| Pi-hole | DNS interno | Resuelve los nombres internos de los servicios a la dirección del servidor y bloquea publicidad y rastreadores |
| Tailscale | VPN | Da acceso a la red de casa desde fuera sin abrir puertos en el router |
| Nginx Proxy Manager | Proxy inverso | Recibe todas las peticiones web, las envía al contenedor que toca según el nombre y gestiona los certificados |
| Panel de actualización (previsto) | Despliegue | Aplica los cambios de este repositorio en el servidor con un botón por stack |
| Portainer | Interfaz de Docker | Ver el estado, los logs y la consola de los contenedores |

Todo corre sobre **Docker**: cada servicio (familiar, domótico o de infraestructura) tiene su propio contenedor.

[← Volver al índice](../README.md)
