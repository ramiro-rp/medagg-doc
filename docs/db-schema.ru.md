# Database Schema (MVP)

Этот документ фиксирует схему базы данных проекта. Сейчас существуют **две версии описания схемы**:

1) **DBA init SQL / ER‑модель** (см. `init-db.sql` и документацию DBA): отдельная Postgres‑схема с таблицами `roles`, `users`, `datasets`, а также дополнительными таблицами `collections` и `search_queries`.

2) **Backend (Django) модели + миграции**: backend API реализован на Django/DRF и использует свои модели и миграции (включая таблицы `django.contrib.auth`).

Поскольку эти источники пока не полностью согласованы, ниже они разделены и отмечены известные расхождения.

---

## A) DBA‑схема (init-db.sql)

### Основные таблицы

- `roles`
- `users`
- `anatomical_areas`
- `modalities`
- `ml_tasks`
- `tags`
- `datasets`

### Таблицы связей many‑to‑many

- `dataset_modalities`
- `dataset_ml_tasks`
- `dataset_tags`

### Коллекции и сохранённые поиски

- `collections`
- `collection_datasets`
- `search_queries`
- `search_query_datasets`

> Примечание: в текущих материалах DBA **нет** колонки `readme_content` в таблице датасетов.

---

## B) Backend‑схема (Django)

Backend API использует Django‑модели и миграции. В snapshot backend, который используется для обновления документации, serializer датасета включает поле `license`, и в репозитории есть миграция для этого поля.

### Важное замечание по согласованию: `readme_content`

Генератор README для датасетов был реализован в feature‑ветке (`feature/search`) и использует поле с контентом README (обычно `readme_content`).

- В текущем DBA init SQL / ER‑описании этого поля нет.
- Наличие этой колонки в развернутой Postgres‑схеме нужно подтвердить у backend‑разработчика и DBA.
