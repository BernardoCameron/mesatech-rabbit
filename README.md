# MesaTech Cloud: RabbitMQ

RabbitMQ del proyecto MesaTech Cloud (EP2, DSY1107), instalado con Docker Compose en su propia EC2.

Este repo nace de separar la mensajeria del backend. Por ahora solo tiene la estructura; el
`compose.yaml` y las definiciones se agregan en los siguientes commits.

## Lo que va a tener

- `compose.yaml`: imagen oficial con etiqueta `-management`, hostname fijo, puertos 5672 y 15672,
  volumen persistente en `/var/lib/rabbitmq` y politica de reinicio.
- `definitions.json`: virtual host `mesatech`, usuarios `catalogo_pub` y `solicitudes_con`, exchange
  `catalogo.exchange` (direct, durable), cola `solicitudes.catalogo.queue` y binding `categoria.creada`.
- `.env.example` con valores falsos. Las credenciales reales van en `.env` (no se sube).
