# Обзор API (MVP)

> Это **высокоуровневое** описание. Источником истины являются схема и код
> в `medagg-backend` (Django + DRF).

Базовый URL backend-а: `http://<server-host>`

Далее предполагается, что все эндпоинты находятся под префиксом `/api`,
например: `GET /api/datasets/`, `POST /api/datasets/search/`. При желании
можно добавить версионирование (`/api/v1`).

---

## 1. Аутентификация

### POST `/api/auth/login/`

Аутентифицирует пользователя и возвращает access-токен (например, JWT)
или устанавливает сессионную cookie.

Пример запроса:

```json
{
  "username": "user@example.com",
  "password": "string"
}
```

Пример ответа (JWT-стиль):

```json
{
  "access_token": "jwt-token-here",
  "token_type": "bearer"
}
```

Затем токен передаётся в заголовке:

```http
Authorization: Bearer <token>
```

---

## 2. Каталог датасетов

### GET `/api/datasets/`

Возвращает список датасетов с основными полями, необходимыми фронтенду.

Опциональные query-параметры:

- `q` – строка полнотекстового поиска;
- `modality` – код/ID модальности;
- `area` – код/ID анатомической области;
- `task_type` – код/ID типа ML‑задачи;
- `tags` – список ID или имён тегов (через запятую).

Пример ответа (упрощённо, один элемент):

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

Это соответствует форме объекта датасета, которую ожидает фронтенд:

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

Возвращает подробную информацию по одному датасету.

Структура совпадает с ответом `GET /api/datasets/`, при необходимости
может быть более развёрнутое поле `description` и дополнительные метаданные.

---

## 3. Поиск

Поиск вынесен в отдельный эндпоинт, который фронтенд использует для запросов
по ключевым словам и структурированным фильтрам.

### POST `/api/datasets/search/`

Выполняет поиск по текстовому запросу и фильтрам.

Пример тела запроса:

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

Формат ответа:

- список объектов датасетов с **той же структурой**, что и в
  `GET /api/datasets/` (отдельная обёртка для MVP не требуется).

Пример (упрощённо):

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

(Внутри поисковый модуль может вычислять оценку релевантности, но для MVP
её необязательно отдавать отдельным полем.)

---

## 4. Пользовательские коллекции

### POST `/api/collections/`

Создаёт новую **коллекцию** – запрос на формирование пользовательского набора
(архива ZIP) для текущего пользователя.

Пример запроса:

```json
{
  "query": "pneumonia chest xray",
  "datasetIds": [1, 3],
  "limitPerDataset": 1000
}
```

Пример ответа:

```json
{
  "id": 42,
  "status": "pending",
  "createdAt": "2025-01-01T12:00:00Z"
}
```

### GET `/api/collections/`

Возвращает список коллекций текущего пользователя.

### GET `/api/collections/{id}/`

Возвращает детали коллекции:

- идентификатор;
- текст запроса / краткое описание;
- время создания;
- статус (`pending`, `ready`, `failed`, `expired`);
- размер архива (если доступен);
- ссылка для скачивания (если архив готов).

### GET `/api/collections/{id}/download/`

Скачивает ZIP‑архив, если коллекция готова и не просрочена.

---

## 5. Админские эндпоинты (примеры)

Доступны только администраторам. Возможные примеры:

- `POST /api/admin/datasets/` – создание записи о датасете;
- `PUT /api/admin/datasets/{id}/` – обновление датасета;
- `GET /api/admin/requests/` – просмотр пользовательских запросов и коллекций.

Точный список зависит от конкретной реализации в `medagg-backend`.

---

## 6. Схема и интерактивная документация

Django REST Framework может предоставлять:

- страницы browsable API;
- схему OpenAPI (JSON/YAML) через `drf-spectacular`, `drf-yasg` и т.п.

Для точного описания форматов запросов и ответов используйте опубликованную
схему backend-а или просматривайте viewset-ы и сериализаторы в исходном коде.
