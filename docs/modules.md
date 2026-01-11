# System Modules

This document gives a high‑level overview of the main modules in the system
and how they relate to each other.

The implementation is split across separate repositories; for **low‑level technical details,
installation and developer instructions**, always refer to the README of the corresponding repo.

---

## 0. Repositories

- **Backend API** – `medagg-backend` (Django + DRF)  
  Main docs: `README.md` in that repo (API endpoints, settings, docker-compose).

- **Frontend** – `medagg-frontend` (Vite + React + Ant Design)  
  Main docs: `README.md` in that repo (dev server, build, environment vars).

- **Search module** – `medagg-search`  
  In the current MVP backend snapshot, the search implementation is also vendored under
  `src/libs/medsearch/` and is used by the backend search service.

---

## 1. Backend (`medagg-backend`)

### 1.1 Responsibilities (MVP)

- expose REST API for:
  - dataset catalog (list, detail);
  - search endpoint used by the frontend;
  - vocabulary/dictionary entities (areas, modalities, ML tasks, tags) as part of dataset payloads;
- store data in PostgreSQL;
- provide admin interface for maintaining reference entities and datasets.

### 1.2 Key internal modules

- **Datasets app** (`apps.datasets`)
  - Django models: `Dataset`, `AnatomicalArea`, `Modality`, `MLTask`, `Tag` (+ M2M tables).
  - DRF viewsets/serializers under `apps.datasets.api.v1`.
- **Search app** (`apps.search`)
  - Search API under `apps.search.api.v1`.
  - Search service integrates `libs.medsearch`.
- **README generator** (part of `apps.datasets`)
  - Generates markdown README content for a dataset.
  - Output is stored in `Dataset.readme_content`.
  - In the reference MVP branch, README is generated automatically on save if `readme_content` is empty.

---

## 2. Frontend (`medagg-frontend`)

### 2.1 Responsibilities (MVP)

- provide a SPA UI for:
  - dataset list (catalog)
  - dataset detail page
  - search input + results
  - viewing generated dataset README content

### 2.2 Backend integration

Frontend reads `VITE_API_URL` (base URL for backend API, e.g. `http://127.0.0.1:8000/api/v1`)
and uses:

- `GET /datasets`
- `GET /datasets/{id}`
- `POST /search/datasets`

---

## 3. Database (PostgreSQL)

The MVP schema is centered around the dataset catalog and reference dictionaries.
See `db-schema.md` for entities and relations.

> Note: the backend uses Django migrations as the standard schema management mechanism.
> If the team applies DB changes manually (SQL/ALTER), reflect them in both migrations and docs to avoid drift.
