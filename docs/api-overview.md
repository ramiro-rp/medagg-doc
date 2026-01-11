# API Overview (MVP)

> This is a **high‑level** description of the MVP API contract. The source of truth
> is the backend code (`medagg-backend`, Django + DRF).

## Base URL

Backend API base: `http://<server-host>:8000/api/v1/`

Main resources:

- `datasets/` – dataset catalog (read-only)
- `search/` – search endpoints

---

## 1. Authentication (MVP)

The MVP API is typically used **without** token-based authentication.

- Admin UI: `GET /admin/` (Django admin)
- Browsable API login (dev only): `GET /api-auth/`

If authentication is required later (JWT/OAuth), this document should be updated
together with the backend implementation.

---

## 2. Dataset catalog

### GET `/api/v1/datasets/`

Returns a list of datasets (detailed serializer is used for list & detail).

Response example (single item, simplified):

```json
[
  {
    "id": 1,
    "title": "Test Brain MRI Dataset",
    "description": "Test dataset for README generation",
    "external_path": "https://example.com/datasets/brain-mri",
    "local_path": null,
    "record_count": 100,
    "size": 500,
    "anatomical_area": 1,
    "anatomical_area_name": "Brain",
    "modalities": [{"id": 1, "name": "MRI"}],
    "ml_tasks": [{"id": 1, "name": "Segmentation"}],
    "tags": [{"id": 1, "name": "medical"}],
    "created_at": "2026-01-11T12:00:00Z",
    "updated_at": "2026-01-11T12:00:00Z",
    "readme_content": "# Dataset README\n..."
  }
]
```

Notes:

- Field naming uses **snake_case**.
- `readme_content` is generated server‑side by the README generator module.
  If it is empty in your environment, verify the DB schema and generation workflow.

### GET `/api/v1/datasets/{id}/`

Returns a single dataset (same schema as list items).

---

## 3. Dataset search

### POST `/api/v1/search/datasets/`

Performs a search over datasets.

Request body (minimal):

```json
{ "query": "brain mri" }
```

Optional filters can be passed as **query parameters**:

- `anatomical_area_name`
- `record_count_min`, `record_count_max`
- `size_min`, `size_max`
- `modalities_list` (list of strings)
- `ml_tasks_list` (list of strings)
- `tags_list` (list of strings)
- `ordering` (backend-defined ordering key)

Response:

```json
{
  "count": 12,
  "results": [ /* datasets in the same shape as /datasets/ */ ]
}
```

---

## 4. Status codes (typical)

- `200 OK` – successful request
- `400 Bad Request` – validation errors (e.g. too short query)
- `404 Not Found` – dataset not found (detail endpoint)
- `500` – server error (check backend logs)
