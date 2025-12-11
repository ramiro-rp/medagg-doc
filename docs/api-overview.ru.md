# Обзор API (MVP)

> Это **высокоуровневое** описание. Источником истины являются схема и код
> в `medagg-backend` (Django + DRF).

Базовый URL backend-а: `http://<server-host>/api/v1`

---

## 1. Аутентификация

### POST `/auth/login/`

Аутентифицирует пользователя и возвращает токен доступа (например, JWT)
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

Далее токен передаётся в заголовке:

```http
Authorization: Bearer <token>
```

---

## 2. Каталог наборов данных

### GET `/datasets/`

Возвращает список наборов данных с возможностью фильтрации.

Типичные параметры запроса:

- `q` – строка свободного текстового поиска;
- `modality` – код / ID модальности;
- `area` – код / ID анатомической области;
- `task_type` – код / ID типа ML-задачи;
- `tags` – список ID или имён тегов (через запятую).

Пример ответа (упрощённо):

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

Возвращает детальную информацию по одному набору данных.

---

## 3. Поиск

Поиск может быть реализован:

- через дополнительные параметры `GET /datasets/`; или
- через отдельный эндпоинт.

Пример отдельного эндпоинта:

### POST `/search/datasets/`

Выполняет поиск по свободному тексту и структурированным фильтрам.

Пример запроса:

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

Пример ответа (упрощённо):

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

## 4. Коллекции пользователя

### POST `/collections/`

Создаёт новую **коллекцию** – запрос на формирование кастомного набора данных.

Пример запроса:

```json
{
  "query_text": "pneumonia chest xray",
  "dataset_ids": [1, 3],
  "limit_per_dataset": 1000
}
```

Пример ответа:

```json
{
  "id": 42,
  "status": "pending",
  "created_at": "2025-01-01T12:00:00Z"
}
```

### GET `/collections/`

Возвращает список коллекций для текущего пользователя.

### GET `/collections/{id}/`

Возвращает детали коллекции:

- идентификатор;
- текст запроса / краткое описание;
- время создания;
- статус (`pending`, `ready`, `failed`, `expired`);
- размер архива (если сформирован);
- ссылка для скачивания (если применимо).

### GET `/collections/{id}/download/`

Скачивание ZIP-архива, если коллекция готова и срок её хранения не истёк.

---

## 5. Админ-эндпоинты (примеры)

Доступны только пользователям с ролью Admin. Возможные примеры:

- `POST /admin/datasets/` – создание записи набора данных;
- `PUT /admin/datasets/{id}/` – редактирование;
- `GET /admin/requests/` – просмотр запросов и коллекций пользователей.

Точный список зависит от реализации в `medagg-backend`.

---

## 6. Схема и интерактивная документация

Django REST Framework может предоставлять:

- страницы browsable API;
- схему OpenAPI (JSON/YAML) через `drf-spectacular`, `drf-yasg` и т.п.

Для точного описания форматов запросов и ответов используйте опубликованную
схему backend-а или просматривайте viewset-ы и сериализаторы в исходном коде.
