# databases-docker-build

Local development database stack powered by Docker Compose.

This repository starts **MySQL 8**, **Redis**, and **PostgreSQL 18 (RC)** with:
- fixed ports exposed to your host machine
- persistent data stored under `./.db/` (ignored by git)
- a shared bridge network named `docker-net`

## What's included

- **MySQL 8**: `mysql:8.0`
  - **Host port**: `3306`
  - **Container name**: `db-mysql8`
  - **Data volume**: `./.db/mysql8` → `/var/lib/mysql`
  - **Auth plugin**: `mysql_native_password` (for compatibility with older clients)
  - **Credentials**:
    - user: `apr_user`
    - password: `password`
    - root password: `password`

- **Redis**: `redis:latest`
  - **Host port**: `6379`
  - **Container name**: `db-redis`
  - **Data volume**: `./.db/redis` → `/data`
  - **Config**: `./config/redis.conf` → `/etc/redis/redis.conf`
  - **Password**: `password` (set via `requirepass` in `config/redis.conf`)

- **PostgreSQL 18 (RC)**: `postgres:18rc1`
  - **Host port**: `5432`
  - **Container name**: `db-postgres18`
  - **Data volume**: `./.db/postgres18` → `/var/lib/postgresql`
  - **Credentials**:
    - user: `postgres`
    - password: `password`

## Prerequisites

- Docker Desktop (or Docker Engine)
- Docker Compose v2 (`docker compose ...`)

## How to build / run

Start all databases in the background:

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

Tail logs (optional):

```bash
docker compose logs -f
```

Stop containers:

```bash
docker compose down
```

Stop containers **and remove data** (destructive):

```bash
docker compose down
rm -rf ./.db
```

## How this repo works

Everything is defined in `docker-compose.yaml`:

- **Services**: `db-mysql8`, `db-redis`, `db-postgres18`
- **Ports**: each service maps a standard DB port from container → host so you can connect using `localhost:<port>`
- **Persistence**: each DB stores data on your machine under `./.db/...` so data survives container restarts/recreates
- **Network**: all containers join the same bridge network:
  - network name: `docker-net`
  - driver: `bridge`
  This lets other containers (or your app containers) connect using service/container names like `db-mysql8`, `db-redis`, `db-postgres18`.

## How to connect

### MySQL

- **Host**: `127.0.0.1`
- **Port**: `3306`
- **User**: `apr_user`
- **Password**: `password`

Example:

```bash
mysql -h 127.0.0.1 -P 3306 -u apr_user -p
```

### Redis

- **Host**: `127.0.0.1`
- **Port**: `6379`
- **Password**: `password`

Example:

```bash
redis-cli -h 127.0.0.1 -p 6379 -a password ping
```

### PostgreSQL

- **Host**: `127.0.0.1`
- **Port**: `5432`
- **User**: `postgres`
- **Password**: `password`

Example:

```bash
psql "postgresql://postgres:password@127.0.0.1:5432/postgres"
```

## Configuration notes

- **Default passwords**: this repo uses the literal password `password` for convenience. Change it before sharing screenshots, running on shared networks, or using it for anything beyond local dev.
- **Redis config file**: Redis loads settings from `config/redis.conf`. If you change the password, update the `requirepass` value in that file.
- **PostgreSQL version**: `postgres:18rc1` is a release candidate image. If you prefer stable, switch to a stable tag in `docker-compose.yaml`.