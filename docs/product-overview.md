# Product Overview – Medical Imaging Dataset Aggregator

## 1. Purpose

Researchers and clinicians often need **custom medical imaging datasets**
for training and evaluating machine learning models. Such data is scattered
across different platforms (Kaggle, PhysioNet, open clinical repositories, etc.)
and uses different formats and structures.

**Medical Imaging Dataset Aggregator** is a web application that:

- centralizes **metadata** about open datasets;
- provides a **single search interface**;
- allows the user to **build and download a custom dataset** according to
  clinical and technical criteria.

In the MVP, the system is focused on **thoracic imaging**
(chest X‑ray and CT), but the architecture is extensible to other
anatomical areas and modalities.

---

## 2. Target audience

- 🧑‍⚕️ **Clinicians** (radiologists, pulmonologists) involved in research;
- 🧪 **ML engineers / data scientists** working with medical images;
- 🎓 **Students** studying ML and medical imaging.

We assume basic technical literacy
(working with ZIP archives and CSV files), but not necessarily programming skills.

---

## 3. Main user goals

1. **Find datasets** relevant to a clinical or research question.
2. **Filter** datasets by modality, anatomical area, ML task type and tags.
3. **Merge data from several sources** into a single consistent collection.
4. **Download** images and a **metadata table** ready for use
   in ML workflows.
5. **Reproduce** previously built collections using the query history.

---

## 4. Key functionality (MVP scope)

### 4.1 Dataset catalog

The system maintains a centralized **dataset catalog** in PostgreSQL.

For each dataset it stores at minimum:

- identifier, title and short description;
- link to the external source;
- anatomical area (e.g. chest);
- modality (X‑ray, CT, MRI, etc.);
- ML task type (classification, segmentation, detection, etc.);
- number of studies / images;
- approximate size;
- tags (e.g. “pneumonia”, “COVID‑19”, “fracture”);
- path to a local copy (if present);
- created / updated timestamps.

### 4.2 Search and filtering

Search over the catalog supports:

- **free‑text query** (keywords in Russian or English);
- **structured filters**:
  - modality;
  - anatomical area;
  - ML task type;
  - tags.

In the result list, each dataset shows:

- short description and source;
- basic metrics (number of images, size);
- relevance score (if the search module is used);
- controls to select datasets for collection building.

### 4.3 Building a collection

Based on search results, the user can request a **custom collection**:

1. The system selects studies / images that match the query and filters.
2. For supported sources, the system:
   - downloads or accesses the required files;
   - stores them in local file storage;
   - creates a **ZIP archive** containing:
     - image files;
     - a **CSV file** with metadata (ID, labels, modality, source dataset, etc.).
3. The system logs the **user query** and links it to the created collection.

### 4.4 Account and history

After authentication, the user has access to a personal area where they can:

- view **history of their collections**;
- see collection status (in progress / ready / expired);
- download ready archives;
- re‑build expired collections if implemented.

### 4.5 Admin functions (minimal set)

An administrator can:

- create / edit / deactivate dataset entries;
- manage vocabularies (modalities, anatomical areas, task types, tags);
- view logs and user requests.

---

## 5. Architecture overview

MVP architecture consists of:

- **Backend** – Django + Django REST Framework  
  REST API for authentication, catalog, search, collections and admin operations,
  with PostgreSQL as the main database.

- **Search module** – separate Python module `medagg-search`  
  Parses free‑text queries (RU/EN) using a YAML taxonomy, can compute relevance
  and extract structured slots (modality, area, pathology, task type).

- **Frontend** – Vite + React + Ant Design  
  Single Page Application (SPA) that communicates with the backend via REST API.

---

## 6. Environment and deployment (MVP)

The MVP is designed to run on **a single Linux host** using Docker:

- backend container (Django + DRF);
- PostgreSQL container;
- optionally a container or service to serve the built frontend;
- local file storage for ZIP archives.

The host may be:

- a developer’s laptop;
- a simple server or VPS;
- a compact device (e.g. Raspberry Pi 4 with SSD) for demo purposes.

Details are described in `deploy-guide.md`.

---

## 7. Out‑of‑scope (beyond MVP)

Possible future directions (not in MVP scope):

- rich “dataset profiles” with extended metadata;
- user‑submitted datasets and moderation;
- recommendation engine based on query and download history;
- integration with external annotation tools;
- distributed / cloud deployment with scaling and full monitoring.
