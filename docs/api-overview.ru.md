# API Overview (MVP)

<<<<<<< Updated upstream
Этот документ описывает **текущую** API‑поверхность, которую использует MVP‑frontend и которая реализована в backend (Django REST Framework).

## Base URL

Все endpoints ниже доступны под:
=======
> Это **высокоуровневое** описание. Источник истины — код в репозитории
> `medagg-backend` (Django + DRF).

## Базовый URL

Для референсного снапшота backend API версионирован:

- Base: `http://<server-host>:8000/api/v1/`

Frontend использует `VITE_API_URL` и затем добавляет пути вроде `/datasets` и
`/search/datasets`.

Контракт, проверенный в интеграционном тесте:

- `VITE_API_URL` задаётся **без завершающего слэша**, например: `http://127.0.0.1:8000/api/v1`
- Эндпойнты в бэкенде принимают пути **со слэшем на конце**: `/datasets/`, `/datasets/{id}/`, `/search/datasets/`
- Если в Django включён `APPEND_SLASH=True`, то `POST /search/datasets` (без слэша) может приводить к 500. Варианты: (1) всегда вызывать эндпойнт со слэшем (как делает фронтенд), или (2) выставить `APPEND_SLASH=False` на бэкенде.
>>>>>>> Stashed changes

- `/api/v1/`

<<<<<<< Updated upstream
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
=======
## 1. Датасеты

### GET `/datasets/`

Возвращает список датасетов с детальными метаданными.

**Ответ (200)**  
JSON-массив объектов датасетов.

### GET `/datasets/{id}/`

Возвращает один датасет по `id`.

**Ответ (200)**  
Объект датасета.
>>>>>>> Stashed changes

Возвращает один датасет по id (тот же набор полей).

<<<<<<< Updated upstream
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
=======
## 2. Поиск

### POST `/search/datasets/`

Поиск датасетов по строке запроса и (опционально) фильтрам.

- Тело запроса (JSON): `{ "query": "<string>" }`
- Фильтры передаются через query params (см. ниже).

**Ответ (200)**

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

### 2.1 Фильтры поиска (query params)

Поддерживаются следующие параметры:

- `anatomical_area_name` – строка
- `record_count_min` / `record_count_max` – числа
- `size_min` / `size_max` – числа
- `modalities_list` – список через запятую (например `MRI,CT`)
- `ml_tasks_list` – список через запятую
- `tags_list` – список через запятую
- `ordering` – список максимум из 2 элементов: `[ "<column>", "asc|desc" ]`  
  По умолчанию: `["created_at", "desc"]`

---

## 3. Примечания

- В интеграционном снапшоте `modalities`, `ml_tasks` и `tags` возвращаются как **массив объектов** `{id, name}`.
  Если фронтенду нужны только строки, он может отображать `tag.name`, но контракт API остаётся объектным.
- Поле `size` хранит **байты**. Для внешних источников (например, Kaggle) значение `total_bytes` может превышать 2^31-1,
  поэтому на стороне БД/ORM нужен `BIGINT` (Django: `BigIntegerField`) или ограничение значения перед сохранением.
- Аутентификация для MVP может быть доступна через Django admin и/или DRF session auth (`/api-auth/`).
  Полноценные аккаунты пользователей, «Мои наборы» и коллекции — **цели следующей итерации**.
- Поиск в MVP может работать только по локальному каталогу. Интеграция с внешними источниками (например, Kaggle)
  зависит от наличия настроенных ключей/секретов и режима запуска.
>>>>>>> Stashed changes
