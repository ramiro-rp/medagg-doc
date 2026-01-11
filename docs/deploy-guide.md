# Deploy Guide

This document describes how to deploy **Medical Imaging Dataset Aggregator**
in a simple single‑node environment (MVP scope).

---

## 1. Architecture overview

System components:

- **Backend** – Django + Django REST Framework;
- **Database** – PostgreSQL;
- **Frontend** – Vite + React + Ant Design (SPA);
- **File storage** – local filesystem (optional, if datasets are mirrored locally).

The MVP can be deployed either:

- as **separate containers** (recommended for demo / single node), or
- with the frontend served separately (dev server) pointing to the backend API.

---

## 2. Requirements

- Linux server or workstation (WSL2 / macOS can be used for development);
- Docker and docker compose (or Docker + compose plugin);
- Git (if cloning repos directly).

---

## 3. Backend deployment (Docker, recommended)

### 3.1 Clone backend

```bash
git clone <backend-repo-url>
cd medagg-backend
```

### 3.2 Environment files

The `docker-compose.yml` expects two env files:

- `./.env/database.env`
- `./.env/settings.env`

If your repo does not include templates for them, create the folder and start from this minimal example.

**`.env/database.env`** (example):

```bash
DB_ENGINE=postgresql
DB_HOST=db
DB_PORT=5432
DB_NAME=medagg
DB_USER=medagg
DB_PASSWORD=change-me
```

**`.env/settings.env`** (example):

```bash
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG_MODE=0
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
```

> Adjust values for your environment (especially secrets and allowed hosts).

### 3.3 Start services

If the project provides the helper script, use it:

```bash
python configure.py --docker
```

Or run compose directly:

```bash
docker compose up -d
```

After startup:

- Backend API: `http://<server-host>:8000/api/v1/`
- Django admin: `http://<server-host>:8000/admin/`
- Browsable API login (dev only): `http://<server-host>:8000/api-auth/`

---

## 4. Frontend deployment

### 4.1 Development mode

Run the frontend dev server and point it to the backend API via `VITE_API_URL`, e.g.:

```bash
VITE_API_URL=http://127.0.0.1:8000/api/v1
```

> If you run the frontend dev server on a different origin, you may need to enable CORS on the backend
> or use a dev proxy in Vite. The exact setup depends on your team’s deployment choice.

### 4.2 Production mode (static build)

For a simple single-node demo, the frontend can be built and served by Nginx (see the frontend repository’s README).
