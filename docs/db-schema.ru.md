# Схема базы данных (MVP)

В этом документе описывается **реально используемая MVP‑модель данных**
для *Medical Imaging Dataset Aggregator*.

База данных — **PostgreSQL**. В репозитории также есть ER‑диаграмма:

- `docs/diagrams/er-medagg-core.png`

> Важно: диаграмма может включать запланированные сущности. Здесь описано то,
> что используется текущей MVP‑реализацией backend-а.

---

## 1. Основные сущности

### 1.1 Справочники

Нормализованные справочники, которые используются в датасетах:

- **`datasets_anatomicalarea`** — анатомическая область (например, Brain)
- **`datasets_modality`** — модальность (например, MRI)
- **`datasets_mltask`** — ML‑задача (например, Segmentation)
- **`datasets_tag`** — теги (например, medical)

Обычно поля:

- `id` (PK)
- `name` (unique)

### 1.2 Каталог датасетов

- **`datasets_dataset`** — основная таблица датасетов

Ключевые поля (MVP):

- `title`
- `description` (nullable)
- `external_path` / `local_path` (nullable)
- `record_count` (nullable)
- `size` (nullable)
- `anatomical_area_id` (FK → anatomical area)
- `created_at`, `updated_at`

**README контент**

В backend-е есть модуль генерации README, который сохраняет markdown в поле:

- `Dataset.readme_content` (text)

Примечание по схеме:
- нужно подтвердить у backend/DBA, что колонка `readme_content` присутствует в развёрнутой Postgres‑схеме
  (в MVP reference ветке она есть в Django‑модели).

---

## 2. Связи

### 2.1 Dataset ↔ справочники

- У датасета **одна** анатомическая область (`FK`).
- У датасета **много** модальностей (`M2M` через `datasets_datasetmodality`).
- У датасета **много** ML‑задач (`M2M` через `datasets_datasetmltask`).
- У датасета **много** тегов (`M2M` через `datasets_datasettag`).

---

## 3. Пользователи

Пользователи/права — стандартные таблицы **Django auth** (`auth_user`, `auth_group`, и т.д.).
Для администрирования используется Django admin.

---

## 4. Источник истины

- Django модели: `apps.datasets.models`
- Django миграции: `apps.datasets.migrations`
- Если изменения в БД делаются вручную (SQL/ALTER), обновляйте миграции и этот документ, чтобы не было рассинхрона.
