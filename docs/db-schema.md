# Database Schema (MVP)

This document reflects the **current database schema** from the DBA materials:

- `init-db.sql`
- ER diagram in `docs/diagrams/`

---

## Reference tables

- `roles` (`id`, `name`)
- `anatomical_areas` (`id`, `name`)
- `modalities` (`id`, `name`)
- `ml_tasks` (`id`, `name`)
- `tags` (`id`, `name`)

---

## Main tables

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

## Link tables (many-to-many)

- `dataset_modalities` (`dataset_id` → `datasets(id)`, `modality_id` → `modalities(id)`)
- `dataset_ml_tasks` (`dataset_id` → `datasets(id)`, `ml_task_id` → `ml_tasks(id)`)
- `dataset_tags` (`dataset_id` → `datasets(id)`, `tag_id` → `tags(id)`)

---

## Collections and saved searches

- `user_dataset_collections`
  - `id`
  - `user_id` → `users(id)`
  - `storage_path_hdfs`
  - `archive_size_mb`
  - `created_at`
  - `expires_at` (set by trigger)

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

A trigger `set_expiration_date` sets `expires_at = created_at + 1 day` on insert into `user_dataset_collections`.
