# Database Dev Stack (Docker Compose)

A language-agnostic, ready-to-run local database stack for developers. Bring up MySQL, PostgreSQL, Redis, Mailpit, plus web UIs (phpMyAdmin for MySQL and Adminer for Postgres) with one command. Works for PHP, Node.js, Python, and more.

## Status & Ports

| Service        | Host Port | Internal | UI Link |
|----------------|-----------|----------|---------|
| MySQL          | 3306      | 3306     | phpMyAdmin → http://localhost:8083 |
| PostgreSQL     | 5432      | 5432     | Adminer → http://localhost:8084 |
| Redis          | 6379      | 6379     | — |
| Mailpit (SMTP) | 1025      | 1025     | Mailpit → http://localhost:8025 |
| Mailpit (UI)   | 8025      | 8025     | Mailpit → http://localhost:8025 |
| phpMyAdmin     | 8083      | 80       | http://localhost:8083 |
| Adminer        | 8084      | 8080     | http://localhost:8084 |


## What’s Included

- MySQL 8.0 (persistent storage, custom `my.cnf`, logs)
- PostgreSQL 13 (persistent storage)
- Redis (persistent storage)
- Mailpit (SMTP testing + web inbox)
- phpMyAdmin (MySQL UI)
- Adminer (Postgres UI)
- A shared Docker network (`sail`) so apps can connect by service name

Optional (commented out in `docker-compose.yml`):
- MinIO (S3-compatible object storage) + bucket bootstrap
- A sample Laravel Sail app service

## Quick Start

1. Install Docker Desktop (macOS/Windows/Linux).
2. Copy env sample (if present) and set credentials:

```bash
cp example.env .env
# Edit .env to set DB_USERNAME, DB_PASSWORD, etc.
```

3. Start the stack:

```bash
docker compose up -d
```

4. Stop when done:

```bash
docker compose down
```

## Default Ports

- MySQL: `3306` (overridable via `FORWARD_DB_PORT`)
- PostgreSQL: `5432`
- Redis: `6379` (overridable via `FORWARD_REDIS_PORT`)
- Mailpit SMTP: `1025` (overridable via `FORWARD_MAILPIT_PORT`)
- Mailpit UI: `8025` (overridable via `FORWARD_MAILPIT_DASHBOARD_PORT`)
- phpMyAdmin UI: `8083`
- Adminer UI: `8084`

## Environment Variables

Set these in `.env` (Compose automatically loads it):

- `DB_USERNAME`: database user (used for MySQL and Postgres)
- `DB_PASSWORD`: database password (used for MySQL and Postgres)
- `FORWARD_DB_PORT`: host port for MySQL (default 3306)
- `FORWARD_REDIS_PORT`: host port for Redis (default 6379)
- `FORWARD_MAILPIT_PORT`: host port for SMTP (default 1025)
- `FORWARD_MAILPIT_DASHBOARD_PORT`: host port for Mailpit UI (default 8025)
- Optional MinIO: `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD` (if you uncomment MinIO)

## Service Details

### MySQL (mysql/mysql-server:8.0)
- Hostnames: inside Docker use `mysql`; from host use `localhost`.
- Credentials: `root` + `DB_PASSWORD`, or `DB_USERNAME`/`DB_PASSWORD`.
- Volumes:
  - `./sail-mysql:/var/lib/mysql` (data persistence)
  - `./my.cnf:/etc/mysql/my.cnf` (config)
  - `./logs:/var/log/mysql` (logs)
- Healthcheck: `mysqladmin ping -p${DB_PASSWORD}`.

### PostgreSQL (postgres:13)
- Hostnames: inside Docker use `postgres`; from host use `localhost`.
- Credentials: `POSTGRES_USER=DB_USERNAME`, `POSTGRES_PASSWORD=DB_PASSWORD`.
- Volume: `./sail-postgres:/var/lib/postgresql/data`.

### Redis (redis:alpine)
- Hostnames: inside Docker use `redis`; from host use `localhost`.
- Volume: `./sail-redis:/data`.

### Mailpit (axllent/mailpit)
- SMTP server on `:1025`; web UI on `:8025`.
- Use it as an SMTP sink for dev/testing (captures outbound mails).

### phpMyAdmin
- UI for MySQL at `http://localhost:8083`.
- Configured to talk to the `mysql` service.

### Adminer
- UI for Postgres at `http://localhost:8084`.
- Default server is the `postgres` service.

## Connect From Your App

Use the service names on the Docker network (`sail`) when your app runs in containers, or `localhost` with mapped ports when running on your host.

### PHP (Laravel) `.env`

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=your_db_name
DB_USERNAME=${DB_USERNAME}
DB_PASSWORD=${DB_PASSWORD}

REDIS_HOST=redis
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="dev@example.com"
MAIL_FROM_NAME="Local Dev"
```

### Node.js

MySQL (mysql2):
```js
const mysql = require('mysql2/promise');
const conn = await mysql.createConnection({
  host: 'localhost', // or 'mysql' if inside Docker
  port: 3306,
  user: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
});
```

Postgres (pg):
```js
const { Client } = require('pg');
const client = new Client({
  host: 'localhost', // or 'postgres' if inside Docker
  port: 5432,
  user: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
});
await client.connect();
```

Redis (ioredis):
```js
const Redis = require('ioredis');
const redis = new Redis({ host: 'localhost', port: 6379 });
```

### Python

MySQL (mysqlclient or PyMySQL):
```python
import pymysql
conn = pymysql.connect(host='localhost', port=3306,
                       user='YOUR_USER', password='YOUR_PASS')
```

Postgres (psycopg):
```python
import psycopg
conn = psycopg.connect(host='localhost', port=5432,
                       user='YOUR_USER', password='YOUR_PASS')
```

Redis (redis-py):
```python
import redis
r = redis.Redis(host='localhost', port=6379)
```

## Web UIs

- phpMyAdmin (MySQL): http://localhost:8083
- Adminer (Postgres): http://localhost:8084
- Mailpit: http://localhost:8025

## Data Persistence & Layout

- MySQL data: `./sail-mysql/`
- Postgres data: `./sail-postgres/`
- Redis data: `./sail-redis/`
- MySQL config: `./my.cnf`
- MySQL logs: `./logs/`

Back up or remove these folders to reset data.

## Enable Optional Services

Uncomment in `docker-compose.yml`:
- MinIO stack (`minio`, `createbuckets`) and set `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`.
- The sample `laravel.test` app service if you want an app container.

## Useful Commands

```bash
# Check container status
docker compose ps

# Tail logs for a service
docker compose logs -f mysql

# MySQL CLI (container)
docker compose exec mysql mysql -uroot -p${DB_PASSWORD}

# Redis ping
docker compose exec redis redis-cli ping
```

## CI: Compose Validation

This repository includes a lightweight GitHub Actions workflow that validates the Compose file on pushes and pull requests.

- Workflow: [.github/workflows/compose-validate.yml](.github/workflows/compose-validate.yml)
- It creates a minimal `.env` and runs `docker compose config -q` to catch syntax or interpolation issues early.

## Troubleshooting

- Port in use: change forwarded ports in `.env` (e.g., `FORWARD_DB_PORT`).
- Credentials fail: ensure `.env` is loaded and values are set; recreate containers if needed (`down` then `up -d`).
- Reset data: stop the stack and remove `sail-mysql`, `sail-postgres`, and `sail-redis` folders (this deletes all data).

---

This stack is intended for local and dev use only. Do not expose these services directly to the public internet without hardening.
