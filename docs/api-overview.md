# API Overview (MVP)

> This is a **high‑level** description. The source of truth is the schema
> and code in `medagg-backend` (Django + DRF).

Base backend URL: `http://<server-host>/api/v1`

---

## 1. Authentication

### POST `/auth/login/`

Authenticates a user and returns an access token (e.g. JWT) or sets
a session cookie.

Request example:

```json
{
  "username": "user@example.com",
  "password": "string"
}
```

Response example (JWT‑style):

```json
{
  "access_token": "jwt-token-here",
  "token_type": "bearer"
}
```

The token is then passed in the header:

```http
Authorization: Bearer <token>
```

---

## 2. Dataset catalog

### GET `/datasets/`

Returns a list of datasets with filtering options.

Typical query parameters:

- `q` – free‑text search string;
- `modality` – modality code / ID;
- `area` – anatomical area code / ID;
- `task_type` – ML task type code / ID;
- `tags` – list of tag IDs or names (comma‑separated).

Response example (simplified):

```json
[
  {
    "id": 1,
    "title": "Chest X-ray Pneumonia Dataset",
    "source_url": "https://example.com/datasets/pneumonia",
    "modality": ["XR"],
    "area": "chest",
    "task_types": ["classification"],
    "num_images": 5863,
    "size_mb": 1200,
    "tags": ["pneumonia", "thorax"]
  }
]
```

### GET `/datasets/{id}/`

Returns detailed information about a single dataset.

---

## 3. Search

Search can be implemented:

- via extra parameters to `GET /datasets/`; or
- via a dedicated endpoint.

Example dedicated endpoint:

### POST `/search/datasets/`

Performs search using free text and structured filters.

Request example:

```json
{
  "query_text": "пневмония грудная клетка КТ",
  "filters": {
    "area": "chest",
    "modality": ["CT"],
    "task_type": ["classification"]
  }
}
```

Response example (simplified):

```json
[
  {
    "dataset": {
      "id": 1,
      "title": "Chest CT Pneumonia",
      "modality": ["CT"],
      "area": "chest"
    },
    "relevance_score": 0.93
  }
]
```

---

## 4. User collections

### POST `/collections/`

Creates a new **collection** – a request to build a custom dataset.

Request example:

```json
{
  "query_text": "pneumonia chest xray",
  "dataset_ids": [1, 3],
  "limit_per_dataset": 1000
}
```

Response example:

```json
{
  "id": 42,
  "status": "pending",
  "created_at": "2025-01-01T12:00:00Z"
}
```

### GET `/collections/`

Returns the list of collections for the current user.

### GET `/collections/{id}/`

Returns collection details:

- identifier;
- query text / short description;
- creation time;
- status (`pending`, `ready`, `failed`, `expired`);
- archive size (if available);
- download link (if applicable).

### GET `/collections/{id}/download/`

Downloads the ZIP archive if the collection is ready and not expired.

---

## 5. Admin endpoints (examples)

Available only to admin users. Possible examples:

- `POST /admin/datasets/` – create dataset entry;
- `PUT /admin/datasets/{id}/` – update dataset;
- `GET /admin/requests/` – view user requests and collections.

The exact list depends on the actual implementation in `medagg-backend`.

---

## 6. Schema and interactive docs

Django REST Framework may provide:

- browsable API pages;
- OpenAPI schema (JSON/YAML) via `drf-spectacular`, `drf-yasg`, etc.

For precise request / response formats, use the published backend schema
or inspect viewsets and serializers in the source code.
