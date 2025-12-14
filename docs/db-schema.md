# Database Schema (MVP)

This document describes the **actual SQL schema** for the MVP of
*Medical Imaging Dataset Aggregator*.

The database engine is **PostgreSQL**. The corresponding ER diagram is:

- `docs/diagrams/er-medagg-core.png`

The model works at the **dataset / collection level**: individual images
are not stored as separate rows in the DB; instead, the system tracks
datasets and user-formed collections of datasets.

---

## 1. Overview of main areas

Conceptually the schema covers four areas:

1. **Users and roles**
2. **Reference dictionaries** (anatomical areas, modalities, ML task types, tags)
3. **Dataset catalog** (source datasets and their attributes, including license)
4. **User activity** (search queries and collected datasets)

---

## 2. Users and roles

### 2.1 `roles`

Simple role dictionary.

Main columns:

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(20) UNIQUE NOT NULL` – role name (e.g. `admin`, `user`).

### 2.2 `users`

Application users.

Main columns:

- `id SERIAL PRIMARY KEY`
- `login VARCHAR(50) UNIQUE NOT NULL`
- `password_hash VARCHAR(255) NOT NULL`
- `role_id INTEGER NOT NULL REFERENCES roles(id)`
- `created_at TIMESTAMPTZ DEFAULT NOW()` – registration time.

---

## 3. Reference dictionaries

### 3.1 `anatomical_areas`

Dictionary of anatomical areas (e.g. chest).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(100) UNIQUE NOT NULL`

### 3.2 `modalities`

Dictionary of imaging modalities (X-ray, CT, etc.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

### 3.3 `ml_tasks`

Dictionary of ML task types (classification, segmentation, etc.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

### 3.4 `tags`

Free-form tags describing datasets (pathologies, keywords, etc.).

- `id SERIAL PRIMARY KEY`
- `name VARCHAR(50) UNIQUE NOT NULL`

---

## 4. Dataset catalog

### 4.1 `datasets`

Represents a **source dataset** (e.g. open chest X-ray dataset).

Main columns:

- `id SERIAL PRIMARY KEY`
- `title VARCHAR(500) NOT NULL`
- `description TEXT`
- `external_link VARCHAR(1000)` – URL to the original resource
- `local_storage_path VARCHAR(500)` – path to a local copy, if present
- `record_count INTEGER` – number of records / images (approximate)
- `dataset_size_mb INTEGER` – approximate size in MB
- `license VARCHAR(100)` – short license identifier (e.g. "CC BY 4.0")
- `anatomical_area_id INTEGER REFERENCES anatomical_areas(id)`
- `created_at TIMESTAMPTZ DEFAULT NOW()`
- `updated_at TIMESTAMPTZ DEFAULT NOW()`

### 4.2 `dataset_modalities`

Many-to-many relation between datasets and modalities.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `modality_id INTEGER NOT NULL REFERENCES modalities(id)`
- `PRIMARY KEY (dataset_id, modality_id)`

### 4.3 `dataset_ml_tasks`

Many-to-many relation between datasets and ML task types.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `ml_task_id INTEGER NOT NULL REFERENCES ml_tasks(id)`
- `PRIMARY KEY (dataset_id, ml_task_id)`

### 4.4 `dataset_tags`

Many-to-many relation between datasets and tags.

- `dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE`
- `tag_id INTEGER NOT NULL REFERENCES tags(id)`
- `PRIMARY KEY (dataset_id, tag_id)`

---

## 5. User activity: searches and collections

### 5.1 `user_search_queries`

Stores user search queries and filter snapshots.

- `id SERIAL PRIMARY KEY`
- `user_id INTEGER REFERENCES users(id)` – may be `NULL` for anonymous searches
- `query_text TEXT NOT NULL`
- `filters JSONB` – serialized filters (area, modalities, tasks, tags, etc.)
- `performed_at TIMESTAMPTZ DEFAULT NOW()`

### 5.2 `user_dataset_collections`

Represents a **collection of datasets** built for a user – this is the
“collected dataset” concept from the ER diagram.

- `id SERIAL PRIMARY KEY`
- `user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE`
- `storage_path_hdfs VARCHAR(500) NOT NULL` – path to the resulting archive
- `archive_size_mb INTEGER` – size of the archive
- `created_at TIMESTAMPTZ DEFAULT NOW()`
- `expires_at TIMESTAMPTZ` – when the archive should expire

A trigger automatically sets `expires_at` to 1 day after `created_at`:

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

Links a user collection with the datasets that were included in it.

- `collection_id INTEGER NOT NULL REFERENCES user_dataset_collections(id) ON DELETE CASCADE`
- `dataset_id INTEGER NOT NULL REFERENCES datasets(id)`
- `relevance_score DECIMAL(5,4)` – optional relevance score from search/ML
- `query_id INTEGER NOT NULL REFERENCES user_search_queries(id) ON DELETE CASCADE`
- `PRIMARY KEY (collection_id, dataset_id)`

This table allows us to:

- reconstruct which datasets were included in a given collection;
- see how a particular dataset was used across user collections and queries.

---

## 6. Notes and deviations from the conceptual model

- The MVP schema **does not model individual images** as separate rows;
  instead, it tracks datasets and collections of datasets. Image-level
  details stay in the external sources or local file structure.
- The field name `storage_path_hdfs` suggests potential use of HDFS, but
  in practice it can point to any storage (local filesystem, mounted volume,
  object storage gateway, etc.).
- Automatic expiration of collections is handled via a trigger that sets
  `expires_at = created_at + 1 day`; the application or background jobs
  can periodically clean up expired archives.
