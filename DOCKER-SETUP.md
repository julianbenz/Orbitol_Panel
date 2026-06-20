# Orbitol Panel – Docker Setup

This guide covers deploying Orbitol Panel (based on Paymenter) from your own source on a server using Docker Compose. The included `docker-compose.yml` builds the image locally from your code, so all your changes are included.

## Prerequisites

- Docker + Docker Compose v2 installed on the server
- Port 80 open (or change the port in `docker-compose.yml`)
- All project files copied/cloned to a directory on the server

---

## First-Time Setup

**1. Set secure passwords**

Open `docker-compose.yml` and replace the two placeholder values at the top:

```yaml
MYSQL_PASSWORD: &db-password "CHANGE_ME"       # database password for the app user
MYSQL_ROOT_PASSWORD: "CHANGE_ME_TOO"           # database root password
```

Use strong, random passwords. Save the file.

**2. Build and start all services**

```bash
docker compose up -d --build
```

This builds your app image from the local `Dockerfile`, then starts the database, Redis cache, and the app container. On first start the app will:
- Auto-generate an `APP_KEY` and write it to `.env` in the project directory
- Wait for the database to be ready
- Run database migrations and seeders automatically

The `.env` file is persisted in your project directory (mounted as `/app/var/.env` inside the container) — do not delete it.

**3. Open in browser**

Navigate to `http://<your-server-ip>` (or `https://orbitol.ch` once DNS and a reverse proxy are configured).

The admin account is created during the database seed step. Check the Paymenter docs for default credentials.

---

## Updating After Code Changes

Pull your changes (or edit files directly on the server), then rebuild and restart:

```bash
git pull                          # if using git
docker compose up -d --build      # rebuilds the image and restarts changed containers
```

Migrations run automatically on every startup.

---

## Useful Commands

```bash
# View live logs
docker compose logs -f paymenter

# Stop all containers (data is preserved)
docker compose down

# Stop and remove all data (destructive!)
docker compose down -v

# Open a shell inside the app container
docker compose exec paymenter ash

# Run an Artisan command
docker compose exec paymenter php artisan <command>
```

---

## Reverse Proxy (HTTPS)

The app listens on port 80. For HTTPS on `orbitol.ch`, place a reverse proxy (Nginx, Caddy, or Traefik) in front of it:

- Change the port mapping in `docker-compose.yml` to `"127.0.0.1:8080:80"` so the app is only reachable locally
- Proxy requests from the reverse proxy to `127.0.0.1:8080`
- Handle TLS termination in the reverse proxy

---

## File Structure After First Start

```
project-directory/
├── database/          # MariaDB data (auto-created)
├── storage/           # App storage: logs, uploads (auto-created)
├── themes/            # Active themes
├── extensions/        # Active extensions
├── .env               # Auto-generated on first start – do not delete
└── docker-compose.yml
```
