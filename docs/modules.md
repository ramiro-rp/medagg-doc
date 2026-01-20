# System Modules

This document gives a high‑level overview of the main modules in the system
and how they relate to each other. The actual implementation is split across
separate repositories; for **low‑level technical details, installation and
developer instructions**, always refer to the README of the corresponding repo.

---

## 0. Repositories

- **Backend API** – `medagg-backend`  
  Reference GitHub repo: <https://github.com/b-barsky/medagg-backend>  
  Note: the team may maintain forks/feature branches (e.g., `devel`, `feature/search`, `feature/readme`).
  For integration tests, always follow the README of the exact branch you run.

- **Search module** – `medagg-search`  
  GitHub: <https://github.com/ESBehtev/medagg-search>  
  Note: in the backend, the search code is expected under `src/libs/medsearch/`
  (commonly populated via a git submodule or by copying the `medagg-search` package).

- **Frontend** – `frontend-DataHive`  
  GitHub: <https://github.com/Nikitmen/frontend-DataHive>  
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

- `GET  {VITE_API_URL}/datasets/`
- `GET  {VITE_API_URL}/datasets/{id}/`
- `POST {VITE_API_URL}/search/datasets/` with JSON body `{ "query": "brain mri" }`

---

## 4. README generation (experimental)

An experimental feature branch introduces a `Dataset.readme_content` column
and a generator that can populate it based on dataset metadata.

Current integration-tested scope of the MVP does **not** require this feature
to be enabled. If it is enabled:

- the generated text is stored in `datasets_dataset.readme_content` (nullable text);
- generation is typically triggered on `Dataset.save()` if the field is empty,
  or via an explicit "update" method (implementation-specific);
- the API/serializers must explicitly expose the field if the frontend should show it.

Because this feature touches model code and migrations, it should be merged/rebased
onto the main backend branch carefully to avoid schema drift.

Implementation note from integration tests:
- a `readme_content` DB field exists in the README feature snapshot, but it must not break the existing API contract
  used by the current frontend (e.g., keep `license` and `tags` serializer formats consistent).

