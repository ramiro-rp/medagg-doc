# Deploy Guide

This document describes how to deploy
**Medical Imaging Dataset Aggregator** in a simple single‑node environment.

---

## 1. Architecture overview

System components:

- **Backend** – Django + Django REST Framework;
- **Database** – PostgreSQL;
- **Frontend** – Vite + React + Ant Design (SPA);
- **File storage** – local filesystem for ZIP archives.

In the MVP, all components are deployed on **a single Linux host**, typically
using Docker / docker-compose.

---

## 2. Requirements

- Linux server or workstation (WSL2 / macOS can be used for development);
- Docker and docker-compose (or Docker + compose plugin);
- Git;
- optional for demo: Raspberry Pi 4 or similar + SSD.

---

## 3. Backend deployment (short version)

Typical workflow:

```bash
git clone https://github.com/b-barsky/medagg-backend.git
cd medagg-backend

# Copy example env file if present
cp .env.example .env

# Configure .env (DB connection, secret key, etc.)

# Start services
docker compose up -d  # or: docker-compose up -d
```

After startup:

- backend API is available at `http://<server-host>:8000/api/v1/`;
- browsable API / schema may be available at `/api/`, `/api/schema/`
  (depending on configuration).

Run migrations (if not applied automatically):

```bash
docker compose exec backend python manage.py migrate
```

---

## 4. Frontend deployment (short version)

### Development mode

```bash
git clone https://github.com/solinoid/medagg-frontend.git
cd medagg-frontend

npm install
npm run dev
```

By default the Vite dev server runs at `http://localhost:5173/`
and talks to the backend API.

### Simple production setup

1. Build the frontend:

   ```bash
   npm run build
   ```

2. Serve the `dist/` directory via `nginx`, `caddy` or another HTTP server.

Important:

- configure the correct backend API base URL for the frontend;
- configure CORS on the backend to allow requests from the frontend origin.

---

## 5. Configuration

Key parameters (usually via `.env`):

- **Backend**
  - DB host, port, name, user, password;
  - Django secret key;
  - allowed hosts list;
  - CORS settings;
  - path to the directory with ZIP archives (built collections).
- **PostgreSQL**
  - volume / directory for DB data.
- **Frontend**
  - API base URL (e.g. `VITE_API_BASE_URL`).

See the READMEs in each repo for exact variable names.

---

## 6. Backup and restore

### 6.1 PostgreSQL

Backup example:

```bash
pg_dump -U <db_user> <db_name> > backup.sql
```

Restore example:

```bash
psql -U <db_user> <db_name> < backup.sql
```

In Docker environments, these commands are run via `docker compose exec db ...`
as described in the backend README.

### 6.2 Built archives

Built collections are stored as ZIP files on the host filesystem
or in a mounted Docker volume. For backup you can:

- periodically archive or copy this directory;
- use an external backup system.

If an automatic retention policy is enabled (deleting archives older than N days),
decide in advance whether long‑term storage is required.

---

## 7. Monitoring and logs

Basic monitoring can be done via logs:

```bash
docker compose logs -f backend
docker compose logs -f db
```

For more serious setups you can:

- configure HTTPS via a reverse proxy (nginx, caddy);
- add metrics and dashboards (Prometheus, Grafana, etc.).

---

## 8. MVP deployment limitations

- single‑node environment (no clustering or horizontal scaling);
- only local file storage for built archives;
- minimal monitoring and alerting out of the box.
