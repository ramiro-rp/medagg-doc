# Схема базы данных (снапшот интеграционного теста)

<<<<<<< Updated upstream
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
=======
Этот документ описывает **схему БД, наблюдаемую в ходе интеграционных тестов** MVP backend
(Django ORM + migrations) на **PostgreSQL**.

> В проекте может существовать отдельная DBA-схема (например, `init-db.sql` и ER-диаграммы) для планирования продакшна.
> Перед продакшн-развёртыванием ORM-миграции и DBA SQL-скрипты должны быть синхронизированы.

---

## 1. Область действия

Реализовано в интеграционном снапшоте:

- Справочники: анатомические области, модальности, типы ML-задач, теги.
- Каталог датасетов: таблица датасетов и связи many-to-many.

Запланировано / не реализовано в протестированном MVP снапшоте (оставляем как цели следующей итерации):

- Учетные записи пользователей и роли.
- История поиска / аналитика.
- Пользовательские коллекции, экспорт/архивы.

---

## 2. Справочники

Логические таблицы (реальные имена зависят от Django app labels; ниже приведены типичные):

### 2.1 Анатомические области

- Таблица: `datasets_anatomicalarea`
- Колонки:
  - `id` (auto PK)
  - `name` `varchar(100)` unique

### 2.2 Модальности

- Таблица: `datasets_modality`
- Колонки:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

### 2.3 ML-задачи

- Таблица: `datasets_mltask`
- Колонки:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

### 2.4 Теги

- Таблица: `datasets_tag`
- Колонки:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

---

## 3. Каталог датасетов

### 3.1 Датасеты

- Таблица: `datasets_dataset`
- Колонки:
  - `id` (auto PK)
  - `title` `varchar(500)` (обязательно)
  - `description` `text` (nullable)
  - `external_path` `varchar(1000)` (nullable)
  - `local_path` `varchar(500)` (nullable)
  - `record_count` `integer` (nullable)
  - `size` (nullable)
    - хранится в **байтах**
    - **важно:** для внешних источников (например, Kaggle) `total_bytes` может превышать 2^31-1.
      Для PostgreSQL используйте `BIGINT` (Django: `BigIntegerField`) или ограничивайте значение перед сохранением.
  - `license` `varchar(500)` (разрешён blank; используется фронтендом в протестированном контракте)
  - `anatomical_area_id` FK → `datasets_anatomicalarea(id)` (nullable)
  - `created_at` timestamp
  - `updated_at` timestamp
  - `readme_content` `text` (nullable, **экспериментальная** функция генерации README)

### 3.2 Таблицы связей many-to-many

Реализованы через явные “through” модели в Django:

- Модальности:
  - Таблица: `datasets_datasetmodality`
  - Колонки: `dataset_id` FK, `modality_id` FK
  - Уникальность: (`dataset_id`, `modality_id`)

- ML-задачи:
  - Таблица: `datasets_datasetmltask`
  - Колонки: `dataset_id` FK, `ml_task_id` FK
  - Уникальность: (`dataset_id`, `ml_task_id`)

- Теги:
  - Таблица: `datasets_datasettag`
  - Колонки: `dataset_id` FK, `tag_id` FK
  - Уникальность: (`dataset_id`, `tag_id`)

---

## 4. Примечания

### 4.1 Единицы размера и производительность

Backend хранит **только метаданные** (названия, описания, теги и т.п.).
Он **не скачивает** содержимое датасетов при поиске; сохраняются лишь поля типа `external_path` и `size`.

### 4.2 Кодировка текста

Все текстовые поля должны храниться как Unicode (UTF‑8) end-to-end.
Если генератор README выводит не-ASCII текст (например, русский),
не должно быть лишних `.encode()`/`.decode()` шагов перед записью в БД.
Если в Windows-терминале символы выглядят «битым текстом», проверьте кодовую страницу терминала;
сама БД при корректной записи должна хранить валидный Unicode.
>>>>>>> Stashed changes
