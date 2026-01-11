# Database Schema (MVP)

This document describes the **implemented MVP data model** for
*Medical Imaging Dataset Aggregator*.

The database engine is **PostgreSQL**. The repository also contains an ER diagram:

- `docs/diagrams/er-medagg-core.png`

> Important: the diagram may include planned entities. This document focuses on the entities
> used by the current backend MVP implementation.

---

## 1. Main entities

### 1.1 Reference dictionaries

These tables store normalized dictionaries used across datasets:

- **`datasets_anatomicalarea`** – anatomical area (e.g. Brain)
- **`datasets_modality`** – modality (e.g. MRI)
- **`datasets_mltask`** – ML task (e.g. Segmentation)
- **`datasets_tag`** – free-form tags (e.g. medical)

Each dictionary row typically contains:

- `id` (PK)
- `name` (unique)

### 1.2 Dataset catalog

- **`datasets_dataset`** – the main dataset table

Key fields (MVP):

- `title`
- `description` (nullable)
- `external_path` / `local_path` (nullable; source URL and/or local mirror path)
- `record_count` (nullable)
- `size` (nullable; numeric value used by backend and README generator)
- `anatomical_area_id` (FK → anatomical area)
- timestamps: `created_at`, `updated_at`

**README content**

The backend includes a dataset README generator module that writes generated markdown to:

- `Dataset.readme_content` (text)

Schema note:
- confirm with backend/DBA whether the `readme_content` column is present in the deployed PostgreSQL schema
  (the field is present in the Django model in the MVP reference branch).

---

## 2. Relations

### 2.1 Dataset ↔ dictionaries

- A dataset has **one** anatomical area (`FK`).
- A dataset has **many** modalities (`M2M` through `datasets_datasetmodality`).
- A dataset has **many** ML tasks (`M2M` through `datasets_datasetmltask`).
- A dataset has **many** tags (`M2M` through `datasets_datasettag`).

---

## 3. Users

User accounts and permissions are handled by standard **Django auth** tables
(`auth_user`, `auth_group`, etc.). The MVP backend uses Django admin for management.

---

## 4. Source of truth

- Django models: `apps.datasets.models`
- Django migrations: `apps.datasets.migrations`
- If the team applies manual SQL changes, update both migrations and this document to avoid schema drift.
