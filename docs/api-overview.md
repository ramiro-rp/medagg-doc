# API Overview (MVP)

> This is a **high-level** description. The source of truth is the schema
> and code in `medagg-backend` (Django + DRF).

Base backend URL: `http://<server-host>`

All endpoints below are assumed to live under the `/api` prefix, e.g.
`GET /api/datasets/`, `POST /api/datasets/search/`. Versioning (e.g. `/api/v1`)
can be added later if needed.

---

## 1. Authentication

### POST `/api/auth/login/`

Authenticates a user and returns an access token (e.g. JWT) or sets
a session cookie.

Request example:

```json
{
  "username": "user@example.com",
  "password": "string"
}
```

Response example (JWT-style):

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

### GET `/api/datasets/`

Returns a list of datasets with basic fields required by the frontend.

Optional query parameters:

- `q` – free-text search string;
- `modality` – modality code / ID;
- `area` – anatomical area code / ID;
- `task_type` – ML task type code / ID;
- `tags` – list of tag IDs or names (comma-separated).

Response example (simplified, single item):

```json
[
  {
    "id": 1,
    "title": "Chest X-ray Pneumonia Dataset",
    "description": "Short dataset description...",
    "externalLink": "https://example.com/datasets/pneumonia",
    "recordCount": 5863,
    "datasetSizeMB": 1200,
    "anatomicalArea": "chest",
    "modalities": ["XR"],
    "mlTasks": ["classification"],
    "license": "CC BY 4.0",
    "tags": ["pneumonia", "thorax"],
    "createdAt": "2025-01-01T12:00:00Z"
  }
]
```

This matches the shape requested by the frontend for a dataset card:

```json
{
  "id": null,
  "title": "",
  "description": "",
  "externalLink": "",
  "recordCount": null,
  "datasetSizeMB": null,
  "anatomicalArea": "",
  "modalities": [],
  "mlTasks": [],
  "license": "",
  "createdAt": ""
}
```

### GET `/api/datasets/{id}/`

Returns detailed information about a single dataset.

The structure is the same as in `GET /api/datasets/`, optionally with a
longer `description` and extra metadata fields if needed.

---

## 3. Search

Search is exposed via a dedicated endpoint that the frontend will call
for keyword-based queries and structured filters.

### POST `/api/datasets/search/`

Performs search using free text and optional filters / tags.

Request example:

```json
{
  "query": "pneumonia chest xray",
  "filters": {
    "anatomicalArea": "chest",
    "modalities": ["XR"],
    "mlTasks": ["classification"],
    "tags": ["pneumonia"]
  }
}
```

Response format:

- a list of dataset objects with the **same structure** as
  `GET /api/datasets/` (no extra wrapper is required for the MVP).

Example (simplified):

```json
[
  {
    "id": 1,
    "title": "Chest X-ray Pneumonia Dataset",
    "description": "Short dataset description...",
    "externalLink": "https://example.com/datasets/pneumonia",
    "recordCount": 5863,
    "datasetSizeMB": 1200,
    "anatomicalArea": "chest",
    "modalities": ["XR"],
    "mlTasks": ["classification"],
    "license": "CC BY 4.0",
    "tags": ["pneumonia", "thorax"],
    "createdAt": "2025-01-01T12:00:00Z"
  }
]
```

(Internally, the search engine may compute a relevance score, but for the MVP
it does not need to be returned as a separate field.)

---

## 4. User collections

### POST `/api/collections/`

Creates a new **collection** – a request to build a custom dataset
(zip archive) for the current user.

Request example:

```json
{
  "query": "pneumonia chest xray",
  "datasetIds": [1, 3],
  "limitPerDataset": 1000
}
```

Response example:

```json
{
  "id": 42,
  "status": "pending",
  "createdAt": "2025-01-01T12:00:00Z"
}
```

### GET `/api/collections/`

Returns the list of collections for the current user.

### GET `/api/collections/{id}/`

Returns collection details:

- identifier;
- query text / short description;
- creation time;
- status (`pending`, `ready`, `failed`, `expired`);
- archive size (if available);
- download link (if applicable).

### GET `/api/collections/{id}/download/`

Downloads the ZIP archive if the collection is ready and not expired.

---

## 5. Admin endpoints (examples)

Available only to admin users. Possible examples:

- `POST /api/admin/datasets/` – create dataset entry;
- `PUT /api/admin/datasets/{id}/` – update dataset;
- `GET /api/admin/requests/` – view user requests and collections.

The exact list depends on the actual implementation in `medagg-backend`.

---

## 6. Schema and interactive docs

Django REST Framework may provide:

- browsable API pages;
- OpenAPI schema (JSON/YAML) via `drf-spectacular`, `drf-yasg`, etc.

For precise request / response formats, use the published backend schema
or inspect viewsets and serializers in the source code.
