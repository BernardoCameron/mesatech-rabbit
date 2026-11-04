# MesaTech Cloud: RabbitMQ

Broker de mensajería de MesaTech Cloud (EP2, DSY1107). `ms-catalogo` publica un evento por cada categoría creada y
`ms-solicitudes` lo consume. RabbitMQ 4.1 con Docker Compose, imagen oficial.

## Contenido

| Archivo | Descripción |
|---|---|
| `compose.yaml` | Servicio `rabbitmq` (`rabbitmq:4.1-management`), hostname `mesatech-rabbit`, volumen `rabbitmq_data`, healthcheck |
| `.env.example` | Variables con valores de ejemplo. Las reales van en `.env` (no se versiona) |

## Requisitos

- Docker Engine y Docker Compose v2 (Ubuntu 24.04: paquetes `docker.io` y `docker-compose-v2`).
- Variables: `RABBITMQ_DEFAULT_USER` y `RABBITMQ_DEFAULT_PASS` (administrador). El hostname fijo evita perder los datos
  del volumen al recrear el contenedor.

## Despliegue

```bash
cp .env.example .env
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
```

## Puertos

| Puerto | Uso | Orígenes permitidos (security group) |
|---|---|---|
| 5672 | AMQP | EC2 de ms-catalogo y de ms-solicitudes |
| 15672 | Panel de administración | IP administrativa |
| 22 | SSH | IP administrativa |

## Recursos

Se crean desde el panel de administración (`http://<ip>:15672`).

| Recurso | Valor |
|---|---|
| Virtual host | `mesatech` |
| Usuario publicador | `catalogo_publicador`, sin tags. Configure `^$`, write `^catalogo\.exchange$`, read `^$` |
| Usuario consumidor | `solicitudes_consumidor`, sin tags. Configure `^$`, write `^$`, read `^solicitudes\.catalogo\.queue$` |
| Administrador | usuario del `.env`, permisos `.*` `.*` `.*` sobre `mesatech` |
| Exchange | `catalogo.exchange`, direct, durable |
| Cola | `solicitudes.catalogo.queue`, classic, durable |
| Binding | `catalogo.exchange` hacia la cola, routing key `categoria.creada` |

No se usa el usuario `guest`. El formato del mensaje está en `docs/evento-categoria.md` del repo `mesatech-backend`.

## Operación

```bash
docker compose stop                                  # detener
docker compose up -d                                 # arrancar
docker compose up -d --force-recreate rabbitmq       # recrear conservando el volumen
```

No usar `docker compose down -v`: borra el volumen con los mensajes pendientes y la configuración. Los mensajes se
publican persistentes y sobreviven a un reinicio del broker.

## Versión

Último commit de `main`.
