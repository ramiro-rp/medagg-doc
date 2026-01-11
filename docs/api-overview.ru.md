# Обзор API (MVP)

> Это **высокоуровневое** описание контракта MVP. Источник истины — код backend-а
> (`medagg-backend`, Django + DRF).

## Базовый URL

Базовый URL API: `http://<server-host>:8000/api/v1/`

Основные ресурсы:

- `datasets/` — каталог датасетов (только чтение)
- `search/` — поиск

---

## 1. Аутентификация (MVP)

В MVP обычно используется API **без** токен‑аутентификации.

- Админка: `GET /admin/` (Django admin)
- Логин для browsable API (только для разработки): `GET /api-auth/`

Если позже появится JWT/OAuth, этот документ нужно обновить вместе с backend-реализацией.

---

## 2. Каталог датасетов

### GET `/api/v1/datasets/`

Возвращает список датасетов (для списка и detail используется один и тот же сериализатор).

Пример ответа (1 элемент, упрощённо):

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

Примечания:

- Имена полей — **snake_case**.
- `readme_content` генерируется на стороне backend-а модулем генерации README.

### GET `/api/v1/datasets/{id}/`

Возвращает один датасет (та же схема, что и в списке).

---

## 3. Поиск по датасетам

### POST `/api/v1/search/datasets/`

Выполняет поиск по датасетам.

Тело запроса (минимально):

```json
{ "query": "brain mri" }
```

Дополнительные фильтры можно передать как **query-параметры**:

- `anatomical_area_name`
- `record_count_min`, `record_count_max`
- `size_min`, `size_max`
- `modalities_list` (список строк)
- `ml_tasks_list` (список строк)
- `tags_list` (список строк)
- `ordering` (ключ сортировки, определённый backend-ом)

Ответ:

```json
{
  "count": 12,
  "results": [ /* датасеты в той же форме, что и /datasets/ */ ]
}
```

---

## 4. Типовые коды ответа

- `200 OK` — успешно
- `400 Bad Request` — ошибки валидации (например, слишком короткий query)
- `404 Not Found` — датасет не найден (detail)
- `500` — ошибка сервера (смотрите логи backend-а)
