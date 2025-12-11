# Database Schema (MVP)

> This is a logical overview of the main entities and relations in the MVP.
> The exact schema is defined by the Django models and migrations in
> `medagg-backend`.

## 1. Main entities

### 1.1 Dataset

Represents a **source dataset** (e.g. an open chest X‑ray dataset).

Key fields (conceptual):

- `id` – internal integer identifier;
- `title` – human‑readable name;
- `description` – short description;
- `source_url` – link to the original resource;
- `area` – anatomical area (FK to `AnatomicalArea` or a choice field);
- `modalities` – one‑to‑many relation to `Modality` (or many‑to‑many);
- `task_types` – one‑to‑many relation to `TaskType` (or many‑to‑many);
- `num_images` – approximate number of images / studies;
- `size_mb` – approximate size in MB;
- `tags` – many‑to‑many to `Tag`;
- `local_path` – path to local copy if present;
- `is_active` – whether this dataset is visible in search;
- `created_at`, `updated_at` – timestamps.

### 1.2 Image / Study

Represents a **single image or study** within a dataset.

Key fields (conceptual):

- `id`;
- `dataset` – FK to `Dataset`;
- `source_id` – original ID within the external dataset (if known);
- `file_path` – relative path to the image file (if stored locally);
- `modality` – FK or enum (if needed per‑image);
- `metadata_json` – optional JSON with extra attributes (DICOM tags, etc.);
- `label_set` – link to labels / annotations (see below).

Depending on how much detail you implement in the MVP, you can keep this table
very simple and only store what is required to reconstruct collections.

### 1.3 Labels / Annotations

For classification‑style tasks a simple model can be used, for example:

- `Label` – vocabulary of possible labels (e.g. “pneumonia”, “normal”);
- `ImageLabel` – relation between `Image` and `Label` with an optional
  confidence score or additional metadata.

### 1.4 Vocabularies

Typical vocabulary tables:

- `AnatomicalArea`
- `Modality`
- `TaskType`
- `Tag`

Each usually contains:

- `id`;
- `code` (short stable identifier);
- `name` (display name);
- optional `description`;
- `is_active` flag.

### 1.5 User and collections

User model follows Django’s auth system.

**Collection** – a custom dataset built by the user.

Key fields:

- `id`;
- `user` – FK to User;
- `query_text` – original free‑text query;
- `filter_snapshot` – JSON with filters at the time of creation;
- `status` – `pending`, `ready`, `failed`, `expired`;
- `num_images` – number of images in the collection;
- `archive_path` – path to the ZIP file (if ready);
- `error_message` – optional text for failures;
- `created_at`, `updated_at`.

A linking table (e.g. `CollectionItem`) can relate a collection to individual
images:

- `collection` – FK to Collection;
- `image` – FK to Image;
- optional extra metadata.

---

## 2. Relations (high‑level)

- `Dataset` 1‑N `Image`
- `Dataset` M‑N `Tag`
- `Dataset` M‑N `Modality`
- `Dataset` M‑N `TaskType`
- `Image` M‑N `Label` (via `ImageLabel`)
- `User` 1‑N `Collection`
- `Collection` M‑N `Image` (via `CollectionItem`)

An ER diagram corresponding to this description can be placed in
`docs/diagrams/` (for example, exported from draw.io or similar).
