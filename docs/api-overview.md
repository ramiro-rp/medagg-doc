# API Overview (MVP)

This document describes the **current** API surface used by the MVP frontend and the backend implementation (Django REST Framework).

## Base URL

All endpoints below are served under:

- `/api/v1/`

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

Returns one dataset by id, with the same field set as above.

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
