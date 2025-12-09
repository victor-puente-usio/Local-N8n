# Local n8n (Docker Compose)

Follow this documentation in order for a successful local setup:

1. Read `WSL.md` — Install and configure WSL (Ubuntu) on Windows 11.
2. Read `DOCKER-WSL.md` — Install and configure Docker Desktop integration with WSL.
3. Read this `README.md` — project-specific steps to run n8n with Docker Compose.

This README assumes you already have a working Ubuntu WSL distro and Docker Desktop configured for WSL (per the two files above).

Quick checklist before starting

- WSL Ubuntu is installed and updated (`WSL.md`).
- Docker Desktop is installed and WSL integration enabled for your Ubuntu distro (`DOCKER-WSL.md`).
- You're operating from the Ubuntu WSL shell for the commands below.

Project files
- `compose.yaml` — Docker Compose config for `caddy`, `postgres`, `redis`, `n8n`, and `n8n-worker`.
- `.env` — environment variables (ignored by git). Copy and edit before first run.
- `init-data.sh` — Postgres init script (creates non-root DB user on first startup).

Prepare `.env`

```bash
# from project root inside WSL
cp .env .env.local
# Edit .env.local and set DOMAIN_NAME, SUBDOMAIN, DATA_FOLDER and secrets
```

Start the stack (WSL Ubuntu shell)

```bash
# ensure you're inside the project directory
cd /path/to/Local-N8n

# bring services up in background
docker compose up -d
```

Check status and logs

```bash
# list running containers
docker compose ps

# follow logs for all services
docker compose logs -f

# follow logs for a single service (example: n8n)
docker compose logs -f n8n
```

Recreate or update the stack

```bash
# stop and remove containers
docker compose down

# rebuild and start (useful after changing .env or compose file)
docker compose up -d --build
```

Postgres initialization

- The `init-data.sh` script is mounted into the Postgres container and runs only on first initialization when the `db_storage` volume is empty. It creates the non-root user specified by `.env`.
- To re-run initialization you must remove the Postgres volume (this destroys DB data):

```bash
docker volume rm Local-N8n_db_storage
```

Access n8n

- Open a browser on your Windows host and navigate to `https://${SUBDOMAIN}.${DOMAIN_NAME}` (ensure your host resolves that domain — see `DOCKER-WSL.md` for hosts file advice).

Troubleshooting pointers

- If containers do not start: run `docker compose logs` and inspect errors for `postgres`, `redis`, or `caddy`.
- If Postgres prints authentication errors, check your `.env.local` credentials match the compose environment variables.
- If Caddy is failing to obtain TLS certs for a local-only domain, consider using a hosts file entry and/or a local/internal TLS setup during development.

Useful commands summary

```bash
docker compose up -d         # start services
docker compose ps            # show running services
docker compose logs -f       # stream logs
docker compose down          # stop and remove containers
docker volume ls             # list volumes
```

If you want me to add a small `Makefile` or a one-file checklist that automates common commands, say the word and I will add it.

License: MIT
