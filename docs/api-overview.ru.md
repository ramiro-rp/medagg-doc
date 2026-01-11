# API Overview (MVP)

Этот документ описывает **текущую** API‑поверхность, которую использует MVP‑frontend и которая реализована в backend (Django REST Framework).

## Base URL

Все endpoints ниже доступны под:

- `/api/v1/`

## Аутентификация

- Backend предоставляет стандартный DRF session auth UI: `/api-auth/`.
- Другие механизмы (например, JWT) **не входят** в текущую реализацию MVP.

## Датасеты

### GET `/api/v1/datasets/`

Возвращает список датасетов.

**Ответ (структура)**

Каждый элемент — объект датасета. Набор полей задаётся serializer’ом backend и включает (как минимум):

- `id`, `title`, `description`
- `external_path`, `local_path`
- `record_count`, `size`, `license`
- `anatomical_area`, `anatomical_area_name`
- `modalities`, `ml_tasks`, `tags`
- `created_at`, `updated_at`

### GET `/api/v1/datasets/{id}/`

Возвращает один датасет по id (тот же набор полей).

## Поиск

### POST `/api/v1/search/datasets/`

Поиск датасетов.

**Body**

- `query` (string, min length 2)

**Необязательные GET query params (фильтры)**

- `anatomical_area_name`
- `record_count_min`, `record_count_max`
- `modalities_list` (через запятую)
- `ml_tasks_list` (через запятую)
- `tags_list` (через запятую)
- `size_min`, `size_max`
- `ordering` (параметр‑список; по умолчанию backend использует `["created_at", "desc"]`)

**Ответ**

- `count` (integer)
- `results` (массив объектов датасетов, та же структура что и в detail)

## Users (admin / internal)

### `/api/v1/users/`

User‑endpoints существуют под этим префиксом. В текущем MVP они предназначены для административного/внутреннего использования и могут требовать аутентификации (см. `/api-auth/`).
