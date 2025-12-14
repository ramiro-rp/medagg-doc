# Схема базы данных (MVP)

В этом документе описывается **фактическая SQL-схема** для MVP системы
*Medical Imaging Dataset Aggregator*.

База данных – **PostgreSQL**. ER-диаграмма, соответствующая схеме, хранится в файле:

- `docs/diagrams/er-medagg-core.png`

Модель работает на уровне **наборов данных и коллекций**: отдельные изображения
не хранятся как строки в БД, система отслеживает только наборы и сформированные
из них коллекции.

---

## 1. Общий обзор областей

Логически схема состоит из четырёх блоков:

1. **Пользователи и роли**
2. **Справочники** (анатомические области, модальности, типы ML‑задач, теги)
3. **Каталог наборов данных** (исходные датасеты и их атрибуты, включая лицензию)
4. **Активность пользователей** (поисковые запросы и собранные датасеты)

---

## 2. Пользователи и роли

### 2.1 `roles`

Простой справочник ролей.

Основные поля:

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(20) UNIQUE NOT NULL` – имя роли (например, `admin`, `user`).

### 2.2 `users`

Пользователи приложения.

Основные поля:

- `id SERIAL PRIMARY KEY`
- `login VARCHAR(50) UNIQUE NOT NULL`
- `password_hash VARCHAR(255) NOT NULL`
- `role_id INTEGER NOT NULL REFERENCES roles(id)`
- `created_at TIMESTAMPTZ DEFAULT NOW()` – дата регистрации.

---

## 3. Справочники

### 3.1 `anatomical_areas`

Справочник анатомических областей (например, грудная клетка).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(100) UNIQUE NOT NULL`

### 3.2 `modalities`

Справочник модальностей визуализации (рентген, КТ и т.д.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

### 3.3 `ml_tasks`

Справочник типов ML‑задач (классификация, сегментация и т.п.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

### 3.4 `tags`

Справочник тегов, которыми описываются датасеты (патологии, ключевые слова и т.п.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

---

## 4. Каталог наборов данных

### 4.1 `datasets`

Описывает **исходный набор данных** (например, открытый датасет рентгенов
грудной клетки).

Основные поля:

- `id SERIAL PRIMARY KEY`
- `title VARCHAR(500) NOT NULL`
- `description TEXT`
- `external_link VARCHAR(1000)` – ссылка на исходный ресурс
- `local_storage_path VARCHAR(500)` – путь к локальной копии (если есть)
- `record_count INTEGER` – количество записей / изображений (оценка)
- `dataset_size_mb INTEGER` – примерный размер в МБ
- `license VARCHAR(100)` – краткое обозначение лицензии (например, «CC BY 4.0»)
- `anatomical_area_id INTEGER REFERENCES anatomical_areas(id)`
- `created_at TIMESTAMPTZ DEFAULT NOW()`
- `updated_at TIMESTAMPTZ DEFAULT NOW()`

### 4.2 `dataset_modalities`

Таблица для связи многие‑ко‑многим между датасетами и модальностями.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `modality_id INTEGER NOT NULL REFERENCES modalities(id)`
- `PRIMARY KEY (dataset_id, modality_id)`

### 4.3 `dataset_ml_tasks`

Таблица для связи многие‑ко‑многим между датасетами и типами ML‑задач.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `ml_task_id INTEGER NOT NULL REFERENCES ml_tasks(id)`
- `PRIMARY KEY (dataset_id, ml_task_id)`

### 4.4 `dataset_tags`

Таблица для связи многие‑ко‑многим между датасетами и тегами.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `tag_id INTEGER NOT NULL REFERENCES tags(id)`
- `PRIMARY KEY (dataset_id, tag_id)`

---

## 5. Активность пользователей: поиски и коллекции

### 5.1 `user_search_queries`

Хранит поисковые запросы пользователей и снимок фильтров.

- `id SERIAL PRIMARY KEY`
- `user_id INTEGER REFERENCES users(id)` – может быть `NULL` для анонимных запросов
- `query_text TEXT NOT NULL`
- `filters JSONB` – сериализованные фильтры (области, модальности, задачи, теги и т.д.)
- `performed_at TIMESTAMPTZ DEFAULT NOW()`

### 5.2 `user_dataset_collections`

Описывает **собранный для пользователя датасет** – это аналог сущности
«собранный датасет» / «коллекция» на ER‑диаграмме.

- `id SERIAL PRIMARY KEY`
- `user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE`
- `storage_path_hdfs VARCHAR(500) NOT NULL` – путь к итоговому архиву
- `archive_size_mb INTEGER` – размер архива
- `created_at TIMESTAMPTZ DEFAULT NOW()`
- `expires_at TIMESTAMPTZ` – время истечения срока хранения архива

Срок жизни архива задаётся триггером, который устанавливает `expires_at`
на один день вперёд от `created_at`:

```sql
CREATE OR REPLACE FUNCTION set_expires_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.expires_at = NEW.created_at + INTERVAL '1 day';
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_expiration_date
    BEFORE INSERT ON user_dataset_collections
    FOR EACH ROW
    EXECUTE FUNCTION set_expires_at();
```

### 5.3 `collection_datasets`

Связывает пользовательскую коллекцию с датасетами, которые в неё вошли.

- `collection_id INTEGER NOT NULL REFERENCES user_dataset_collections(id) ON DELETE CASCADE`
- `dataset_id INTEGER NOT NULL REFERENCES datasets(id)`
- `relevance_score DECIMAL(5,4)` – оценка релевантности от поисковой/ML‑модели (опционально)
- `query_id INTEGER NOT NULL REFERENCES user_search_queries(id) ON DELETE CASCADE`
- `PRIMARY KEY (collection_id, dataset_id)`

Эта таблица позволяет:

- восстановить, какие датасеты вошли в конкретную коллекцию;
- анализировать использование датасетов в разных запросах и коллекциях.

---

## 6. Замечания и отличия от концептуальной модели

- В схеме MVP **нет отдельной таблицы изображений** – система оперирует
  наборами данных и коллекциями наборов. Детали на уровне отдельных файлов
  остаются в исходных ресурсах или в структуре локального хранилища.
- Поле `storage_path_hdfs` по названию ориентировано на HDFS, но фактически
  может указывать на любое хранилище (локальный диск, примонтированный том,
  объектное хранилище и т.п.).
- Автоматическое истечение срока хранения реализовано триггером
  `set_expiration_date`, который устанавливает `expires_at = created_at + 1 day`.
  Приложение или служебные скрипты могут периодически удалять просроченные архивы.
