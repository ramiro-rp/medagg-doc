# Database Schema (MVP)

This document captures the database schema used by the project. At the moment there are **two schema descriptions** in circulation:

1) **DBA init SQL / ER model** (see `init-db.sql` and DBA documentation): describes a standalone Postgres schema with tables like `roles`, `users`, `datasets`, plus auxiliary tables like `collections` and `search_queries`.

2) **Backend (Django) models + migrations**: the running backend API is implemented in Django/DRF and defines its own models and migrations (including `django.contrib.auth` tables).

Because these sources are not fully aligned yet, this document separates them and calls out known gaps explicitly.

---

## A) DBA schema (init-db.sql)

### Core tables

- `roles`
- `users`
- `anatomical_areas`
- `modalities`
- `ml_tasks`
- `tags`
- `datasets`

### Many-to-many link tables

- `dataset_modalities`
- `dataset_ml_tasks`
- `dataset_tags`

### Collections and saved searches

- `collections`
- `collection_datasets`
- `search_queries`
- `search_query_datasets`

> Note: The current DBA materials do **not** include a `readme_content` column for datasets.

---

## B) Backend schema (Django)

The backend API uses Django models and migrations. In the backend snapshot used for documentation updates, the dataset serializer includes `license`, and the repository contains a migration for that field.

### Important alignment note: `readme_content`

A README generator for datasets was implemented in a feature branch (`feature/search`) and uses a dataset README content field commonly referred to as `readme_content`.

- This column is **not** present in the current DBA init SQL / ER description.
- Whether it exists in the deployed Postgres schema should be confirmed with the backend developer and DBA.
