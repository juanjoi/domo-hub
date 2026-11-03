# Componentes

La infraestructura se apoya en unas pocas piezas, y cada una tiene una sola responsabilidad.

| Componente | Función | Responsabilidad principal |
| --- | --- | --- |
| Pi-hole | DNS interno | Envía al servidor cualquier nombre del dominio interno, con un registro comodín, y bloquea publicidad y rastreadores |
| Tailscale | VPN | Da acceso a la red de casa desde fuera sin abrir puertos en el router |
| Traefik | Proxy inverso | Recibe todas las peticiones web y las envía al contenedor que toca según el nombre. Cada servicio declara en su stack cómo se publica |
| Panel de actualización (`domo-board`) | Despliegue | Aplica los cambios de este repositorio en el servidor con un botón por stack |
| Portainer | Interfaz de Docker | Ver el estado, los logs y la consola de los contenedores |

Todo corre sobre **Docker**: cada servicio (familiar, domótico o de infraestructura) tiene su propio contenedor.

[← Volver al índice](../README.md)
