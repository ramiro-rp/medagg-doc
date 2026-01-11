# API Overview (MVP)

> This is a **high-level** description. The source of truth is the code in
> `medagg-backend` (Django + DRF).

## Base URL

For the reference backend snapshot, the API is versioned:

- Base: `http://<server-host>:8000/api/v1/`

The frontend uses `VITE_API_URL` and then appends paths like `/datasets` and
`/search/datasets`.

---

## 1. Datasets

### GET `/datasets/`

Returns a list of datasets with detailed metadata.

**Response (200)**
A JSON array of dataset objects.

### GET `/datasets/{id}/`

Returns a single dataset object.

**Response (200)**
A dataset object.

---

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
- `size_min` / `size_max` – integers
- `modalities_list` – comma-separated list (e.g. `MRI,CT`)
- `ml_tasks_list` – comma-separated list
- `tags_list` – comma-separated list
- `ordering` – list with up to 2 items: `[ "<column>", "asc|desc" ]`  
  Default: `["created_at", "desc"]`

---

## 3. Notes

- Authentication for the MVP is currently handled via the Django admin and/or DRF session auth
  (`/api-auth/`). Token-based auth (JWT) can be added later if required.
- Exact field semantics (e.g., what is stored in `external_path` vs `local_path`) should follow the
  backend serializers and models.
