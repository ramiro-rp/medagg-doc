# System Modules

This document gives a high‑level overview of the main modules in the system
and how they relate to each other. The actual implementation is split across
separate repositories; for **low‑level technical details, installation and
developer instructions**, always refer to the README of the corresponding repo.

---

## 0. Repositories

- **Backend API** – `medagg-backend`  
  GitHub: <https://github.com/AnastasiaGladir/medagg-backend>  
  Main docs: `README.md` in that repo (API endpoints, settings, docker-compose).

- **Search module** – `medagg-search`  
  GitHub: <https://github.com/ESBehtev/medagg-search>  
  Note: in the backend, the search code is expected under `src/libs/medsearch/`
  (commonly populated via a git submodule or by copying the `medagg-search` package).

- **Frontend** – `medagg-frontend`  
  GitHub: <https://github.com/NikaAsadli/medagg-frontend>  
  Main docs: `README.md` in that repo (Vite dev server, build, environment vars).

---

## 1. Backend API (Django + DRF)

The backend is a Django application that provides:

- **Datasets API**: list + detail endpoints for dataset metadata.
- **Search API**: query‑based search plus filter parameters.
- **Admin UI**: Django admin for internal management.

Key implementation directories (in `medagg-backend`):

- `src/apps/datasets/` – dataset catalog (models, serializers, views).
- `src/apps/search/` – search endpoint (calls the search service).
- `src/apps/users/` – user API (thin wrapper around Django auth, if enabled).
- `src/config/` – project settings and URL routing.

---

## 2. Search module

Search is exposed via the backend endpoint:

- `POST /api/v1/search/datasets/`

The backend delegates query execution to `SearchService`
(`src/apps/search/services.py`). The low‑level search logic lives in the
`medagg-search` package (expected under `src/libs/medsearch/`).

---

## 3. Frontend

The frontend is a Vite/React application that calls the backend API.

It uses the environment variable:

- `VITE_API_URL` – base API URL (example for local dev: `http://127.0.0.1:8000/api/v1`)

The current frontend calls:

- `GET  {VITE_API_URL}/datasets`
- `GET  {VITE_API_URL}/datasets/{id}`
- `POST {VITE_API_URL}/search/datasets` with JSON body `{ "query": "brain mri" }`

---

## 4. README generation (experimental)

A README generation feature was developed in a backend feature branch and
shared in the team chat (branch example: `medagg-backend/tree/feature/search`).
It is **not part of the reference backend snapshot unless it is merged**.

If/when it is merged, the documentation should be updated to describe:

- where the generated README is stored (DB column name),
- when it is generated (on create/save vs manual),
- and how it is exposed via API (serializer field).

