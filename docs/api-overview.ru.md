# Обзор API (MVP)

> Это **высокоуровневое** описание. Источник истины — код в репозитории
> `medagg-backend` (Django + DRF).

## Базовый URL

Для референсной версии backend API версионирован:

- Base: `http://<server-host>:8000/api/v1/`

Frontend использует `VITE_API_URL` и затем добавляет пути вроде `/datasets` и
`/search/datasets`.

---

## 1. Датасеты

### GET `/datasets/`

Возвращает список датасетов с детальными метаданными.

**Ответ (200)**  
JSON-массив объектов датасетов.

### GET `/datasets/{id}/`

Возвращает один датасет по `id`.

**Ответ (200)**  
Объект датасета.

---

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

- Аутентификация для MVP сейчас реализована через Django admin и/или DRF session auth
  (`/api-auth/`). Токены (JWT) можно добавить позже при необходимости.
- Семантика полей (`external_path`, `local_path` и т.п.) должна соответствовать сериалайзерам и моделям backend.
