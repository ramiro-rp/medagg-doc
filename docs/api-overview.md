# API Overview (MVP)

<<<<<<< Updated upstream
This document describes the **current** API surface used by the MVP frontend and the backend implementation (Django REST Framework).

## Base URL

All endpoints below are served under:
=======
> This is a **high-level** description. The source of truth is the code in
> `medagg-backend` (Django + DRF).

## Base URL

For the reference backend snapshot, the API is versioned:

- Base: `http://<server-host>:8000/api/v1/`

The frontend uses `VITE_API_URL` and then appends paths like `/datasets` and
`/search/datasets`.

Integration-tested contract:

- Use `VITE_API_URL` **without a trailing slash**, e.g. `http://127.0.0.1:8000/api/v1`.
- Use **trailing slashes** on Django endpoints (examples below): `/datasets/`, `/search/datasets/`.
>>>>>>> Stashed changes

- `/api/v1/`

<<<<<<< Updated upstream
## Authentication

- The backend exposes DRF’s session auth UI at `/api-auth/`.
- Other auth mechanisms (e.g., JWT) are **not** part of the current MVP implementation.

## Datasets

### GET `/api/v1/datasets/`

Returns a list of datasets.

**Response (shape)**

Each item is a dataset object. Fields are defined by the backend serializer and include (at least):

- `id`, `title`, `description`
- `external_path`, `local_path`
- `record_count`, `size`, `license`
- `anatomical_area`, `anatomical_area_name`
- `modalities`, `ml_tasks`, `tags`
- `created_at`, `updated_at`

### GET `/api/v1/datasets/{id}/`
=======
## 1. Datasets

### GET `/datasets/`

Returns a list of datasets with detailed metadata.

**Response (200)**
A JSON array of dataset objects.

### GET `/datasets/{id}/`

Returns a single dataset object.

**Response (200)**
A dataset object.
>>>>>>> Stashed changes

Returns one dataset by id, with the same field set as above.

<<<<<<< Updated upstream
## Search

### POST `/api/v1/search/datasets/`

Performs dataset search.

**Body**

- `query` (string, min length 2)

**Optional GET query params (filters)**

- `anatomical_area_name`
- `record_count_min`, `record_count_max`
- `modalities_list` (comma-separated)
- `ml_tasks_list` (comma-separated)
- `tags_list` (comma-separated)
- `size_min`, `size_max`
- `ordering` (list-like param; backend defaults to `["created_at", "desc"]`)

**Response**

- `count` (integer)
- `results` (array of dataset objects, same shape as dataset detail)

## Users (admin / internal)

### `/api/v1/users/`

User endpoints exist under this prefix. In the current MVP these are intended for administrative/internal use and may require authentication (see `/api-auth/`).
=======
## 2. Search

### POST `/search/datasets/`

Searches for datasets by query and optional filter parameters.

- Request body (JSON): `{ "query": "<string>" }`
- Optional filters are passed as query params (see below).

**Response (200)**
```json
{
  "count": 1,
  "results": [
    {
      "id": 1,
      "title": "Test Brain MRI Dataset",
      "description": "Test dataset",
      "external_path": "https://example.org/datasets/brain-mri",
      "local_path": "/data/brain-mri",
      "record_count": 100,
      "size": 500,
      "license": "CC BY 4.0",
      "anatomical_area": 1,
      "anatomical_area_name": "Brain",
      "modalities": [{"id": 1, "name": "MRI"}],
      "ml_tasks": [{"id": 1, "name": "Segmentation"}],
      "tags": [{"id": 1, "name": "medical"}],
      "created_at": "2026-01-01T12:00:00Z",
      "updated_at": "2026-01-01T12:00:00Z"
    }
  ]
}
```

### 2.1 Search filters (query params)

The search endpoint accepts the following optional query params:

- `anatomical_area_name` – string
- `record_count_min` / `record_count_max` – integers
- `size_min` / `size_max` – integers (treat as bytes; values may exceed 32-bit if the backend stores Kaggle `total_bytes`)
- `modalities_list` – comma-separated list (e.g. `MRI,CT`)
- `ml_tasks_list` – comma-separated list
- `tags_list` – comma-separated list
- `ordering` – list with up to 2 items: `[ "<column>", "asc|desc" ]`  
  Default: `["created_at", "desc"]`

---

## 3. Notes

- Authentication for the MVP is currently handled via the Django admin and/or DRF session auth
  (`/api-auth/`). Token-based auth (JWT) can be added later if required.
- In the tested backend snapshot, `modalities`, `ml_tasks`, and `tags` are arrays of **objects**
  (e.g., `[{"id": 1, "name": "MRI"}]`), not plain strings.
- Depending on the backend branch/configuration, the search endpoint may optionally augment results
  from external sources (e.g., Kaggle) and persist them; this is considered an implementation detail
  and should be guarded by configuration in production.
- Exact field semantics (e.g., what is stored in `external_path` vs `local_path`) should follow the
  backend serializers and models.

>>>>>>> Stashed changes
