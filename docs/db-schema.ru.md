# Схема базы данных (MVP)

Этот документ отражает **текущую схему базы данных** из материалов DBA:

- `init-db.sql`
- ER-диаграмма в `docs/diagrams/`

---

## Справочные таблицы

- `roles` (`id`, `name`)
- `anatomical_areas` (`id`, `name`)
- `modalities` (`id`, `name`)
- `ml_tasks` (`id`, `name`)
- `tags` (`id`, `name`)

---

## Основные таблицы

### `users`

- `id`
- `login`
- `password_hash`
- `role_id` → `roles(id)`
- `created_at`

### `datasets`

- `id`
- `title`
- `description`
- `external_link`
- `local_storage_path`
- `record_count`
- `dataset_size_mb`
- `license`
- `anatomical_area_id` → `anatomical_areas(id)`
- `created_at`
- `updated_at`

---

## Таблицы связей (many-to-many)

- `dataset_modalities` (`dataset_id` → `datasets(id)`, `modality_id` → `modalities(id)`)
- `dataset_ml_tasks` (`dataset_id` → `datasets(id)`, `ml_task_id` → `ml_tasks(id)`)
- `dataset_tags` (`dataset_id` → `datasets(id)`, `tag_id` → `tags(id)`)

---

## Коллекции и сохранённые поисковые запросы

- `user_dataset_collections`
  - `id`
  - `user_id` → `users(id)`
  - `storage_path_hdfs`
  - `archive_size_mb`
  - `created_at`
  - `expires_at` (устанавливается триггером)

- `user_search_queries`
  - `id`
  - `user_id` → `users(id)` (nullable)
  - `query_text`
  - `filters` (JSONB)
  - `performed_at`

- `collection_datasets`
  - `collection_id` → `user_dataset_collections(id)`
  - `dataset_id` → `datasets(id)`
  - `relevance_score`
  - `query_id` → `user_search_queries(id)`

Триггер `set_expiration_date` устанавливает `expires_at = created_at + 1 day` при вставке в `user_dataset_collections`.
