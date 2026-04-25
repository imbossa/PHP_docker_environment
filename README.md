# Docker-based PHP Development Environment

This repository provides a local PHP development stack with PHP-FPM, Nginx, MySQL, Redis, MongoDB, and RabbitMQ.

[English](README.md) | [简体中文](README.zh-CN.md)

## Requirements

- Docker Desktop or Docker Engine with Docker Compose v2
- Available host ports: `80`, `443`, `9000`, `3306`, `6379`, `27017`, `5672`, `15672`
- A Docker bridge network for static service IPs

## Quick Start

1. Create or reuse the external Docker network.

   ```bash
   docker network inspect my_com >/dev/null 2>&1 || \
     docker network create -d bridge --subnet 172.28.1.0/24 my_com
   ```

   If Docker reports that the subnet overlaps with an existing network, either reuse that network by setting `NETWORK_EXT_NAME` in `.env`, or choose another subnet and update `IPV4_PREFIX` accordingly.

2. Create a local environment file.

   ```bash
   cp .env.example .env
   ```

3. Edit `.env`.

   - Set `LOCAL_WEB_PATH` to the host directory that should be mounted to `/var/www`.
   - Set strong local passwords for MySQL, Redis, MongoDB, and RabbitMQ.
   - Set `NETWORK_EXT_NAME` to the external Docker network name, for example `my_com`.

4. Start the stack.

   ```bash
   docker compose up -d
   ```

5. Check status and logs.

   ```bash
   docker compose ps
   docker compose logs --tail=100
   ```

## Service Addresses

| Service  | In containers       | Static network IP | Host port |
|----------|---------------------|-------------------|-----------|
| PHP-FPM  | `com_php:9000`      | `172.28.1.11`     | `9000`    |
| Nginx    | `com_nginx:80`      | `172.28.1.12`     | `80/443`  |
| MySQL    | `com_mysql:3306`    | `172.28.1.13`     | `3306`    |
| Redis    | `com_redis:6379`    | `172.28.1.14`     | `6379`    |
| MongoDB  | `com_mongo:27017`   | `172.28.1.16`     | `27017`   |
| RabbitMQ | `com_rabbitmq:5672` | `172.28.1.20`     | `5672`    |

RabbitMQ Management UI is exposed on `http://localhost:15672`.

## Data and Backups

Persistent data is stored under `app/*/db` or `app/*/data`:

- MySQL: `app/mysql/db`
- Redis: `app/redis/db`
- MongoDB: `app/mongo/db`
- RabbitMQ: `app/rabbitmq/data/rabbitmq-4`

Before changing database image major versions, back up the relevant data directory. Existing ad-hoc backups can be placed under `app/backups/`, which is ignored by Git.

## Useful Commands

```bash
# Start or update containers
docker compose up -d

# Stop and remove containers while keeping bind-mounted data
docker compose down

# Rebuild PHP/Nginx images
docker compose build php nginx

# Check health status
docker compose ps

# Follow logs
docker compose logs -f --tail=100

# Validate generated Compose configuration
docker compose config
```

## Troubleshooting

### Docker Hub EOF or timeout

If Docker fails with `EOF`, `i/o timeout`, or `failed to fetch oauth token`, retry the pull/start command after the network recovers:

```bash
docker compose pull
docker compose up -d
```

### External network missing

If Compose reports that the external network does not exist, create it or set `NETWORK_EXT_NAME` to an existing compatible network:

```bash
docker network ls
docker network inspect my_com >/dev/null 2>&1 || \
  docker network create -d bridge --subnet 172.28.1.0/24 my_com
```

### Health checks stay unhealthy

Inspect the logs for the failing service:

```bash
docker compose logs --tail=200 <service>
```

Common causes are incorrect passwords in `.env`, stale database files from an incompatible image version, or a missing external network.

## License

This project is licensed under the [MIT License](LICENSE).
