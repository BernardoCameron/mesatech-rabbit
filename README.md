# MesaTech Cloud: RabbitMQ

RabbitMQ del proyecto MesaTech Cloud (EP2, DSY1107), instalado con Docker Compose (imagen oficial
`rabbitmq:4.1-management`). Este repo nace de separar la mensajeria del backend.

## Que hay

- `compose.yaml`: imagen con etiqueta explicita, hostname fijo `mesatech-rabbit`, puertos 5672 (AMQP) y
  15672 (panel), volumen persistente en `/var/lib/rabbitmq`, `restart: unless-stopped` y healthcheck.
- `.env.example`: variables con valores falsos. Las claves reales van en `.env` (no se sube).

## Instalar y levantar

```bash
cp .env.example .env        # y cambiar las claves
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
```

Diagnostico: `docker compose logs rabbitmq`. Panel: `http://localhost:15672` (en EC2, la IP de la
instancia desde la IP administrativa), con el usuario administrador del `.env`.

## Crear los recursos (a mano, desde el panel)

Con la sesion iniciada como administrador:

1. **Virtual host**: Admin > Virtual Hosts > Add: `mesatech`.
2. **Usuarios**: Admin > Users > Add user, sin tags (sin acceso al panel):
   - `catalogo_publicador` (publicador, clave de `CATALOGO_PUBLICADOR_PASSWORD`).
   - `solicitudes_consumidor` (consumidor, clave de `SOLICITUDES_CONSUMIDOR_PASSWORD`).
3. **Permisos** (en cada usuario, "Set permission", virtual host `mesatech`):

   | Usuario | Configure | Write | Read | Para que |
   |---|---|---|---|---|
   | `catalogo_publicador` | `^$` | `^catalogo\.exchange$` | `^$` | Catalogo publica |
   | `solicitudes_consumidor` | `^$` | `^$` | `^solicitudes\.catalogo\.queue$` | Solicitudes consume |

   Ninguno es administrador ni puede crear o borrar recursos. No se usa `guest`.
   Tambien dar al usuario administrador permisos `.*` `.*` `.*` sobre `mesatech` para ver todo desde el panel.
4. **Exchange** (virtual host `mesatech` > Exchanges > Add): `catalogo.exchange`, tipo `direct`, durable.
5. **Cola** (Queues > Add): `solicitudes.catalogo.queue`, tipo classic, durable, sin auto-delete.
6. **Binding**: en la cola, Bindings > From exchange `catalogo.exchange`, routing key `categoria.creada`.

## Reinicio sin perder datos

```bash
docker compose up -d --force-recreate rabbitmq   # NO usar "down -v": borra el volumen
```

El hostname y el volumen se conservan, asi que siguen el virtual host, el exchange, la cola y los
mensajes pendientes (los mensajes se publican como persistentes).

## En EC2 (Ubuntu 24.04)

Instalar Docker Engine y el plugin Compose, clonar este repo y seguir los mismos pasos.
Security group: 5672 solo desde los security groups de las EC2 de Catalogo y Solicitudes; 15672 y 22 solo
desde la IP administrativa (/32).
