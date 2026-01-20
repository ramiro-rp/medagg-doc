# Database Schema (integration snapshot)

This document describes the **database schema observed during integration tests** of the MVP backend (Django ORM + migrations) on **PostgreSQL**.

> A separate DBA-managed SQL schema (e.g., `init-db.sql` and ER diagrams) may exist for production planning. Before production deployment, the ORM migrations and the DBA SQL scripts must be aligned.

---

## 1. Scope

Implemented in the integration snapshot:

- Reference dictionaries: anatomical areas, modalities, ML task types, and tags.
- Dataset catalog: datasets plus their many-to-many links.

Planned / not implemented in the tested MVP snapshot (keep as goals for the next iteration):

- User accounts and roles.
- Search history / analytics.
- User collections and dataset export/archives.

---

## 2. Reference dictionaries

Logical tables (actual table names depend on Django app labels; typical names are shown):

### 2.1 Anatomical areas

- Table: `datasets_anatomicalarea`
- Columns:
  - `id` (auto PK)
  - `name` `varchar(100)` unique

### 2.2 Modalities

- Table: `datasets_modality`
- Columns:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

### 2.3 ML tasks

- Table: `datasets_mltask`
- Columns:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

### 2.4 Tags

- Table: `datasets_tag`
- Columns:
  - `id` (auto PK)
  - `name` `varchar(50)` unique

---

## 3. Dataset catalog

### 3.1 Datasets

- Table: `datasets_dataset`
- Columns:
  - `id` (auto PK)
  - `title` `varchar(500)` (required)
  - `description` `text` (nullable)
  - `external_path` `varchar(1000)` (nullable)
  - `local_path` `varchar(500)` (nullable)
  - `record_count` `integer` (nullable)
  - `size` (nullable)
    - Stored as **bytes**.
    - **Important:** Kaggle `total_bytes` can exceed 2^31-1. For PostgreSQL, use `BIGINT` (Django: `BigIntegerField`) or cap the value before persisting.
  - `license` `varchar(500)` (blank allowed; used by the frontend contract in the tested snapshot)
  - `anatomical_area_id` FK → `datasets_anatomicalarea(id)` (nullable)
  - `created_at` timestamp
  - `updated_at` timestamp
  - `readme_content` `text` (nullable, **experimental** README generator feature)

### 3.2 Many-to-many relation tables

These are implemented via explicit “through” models in Django:

- Modalities:
  - Table: `datasets_datasetmodality`
  - Columns: `dataset_id` FK, `modality_id` FK
  - Uniqueness: (`dataset_id`, `modality_id`)

- ML tasks:
  - Table: `datasets_datasetmltask`
  - Columns: `dataset_id` FK, `ml_task_id` FK
  - Uniqueness: (`dataset_id`, `ml_task_id`)

- Tags:
  - Table: `datasets_datasettag`
  - Columns: `dataset_id` FK, `tag_id` FK
  - Uniqueness: (`dataset_id`, `tag_id`)

---

## 4. Notes

### 4.1 Size units and performance

The backend stores **metadata only** (titles, descriptions, tags, etc.). It does **not** download dataset payloads during search; it only stores fields like `external_path` and `size`.

### 4.2 Text encoding

All text fields should be stored as Unicode (UTF-8) end-to-end. If a README generator outputs non-ASCII text (e.g., Russian), ensure no extra `.encode()`/`.decode()` steps are applied before writing to the DB. If characters look garbled in a Windows terminal, verify the terminal code page; the DB itself should still store valid Unicode.
