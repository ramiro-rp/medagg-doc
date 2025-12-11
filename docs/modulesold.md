# System Modules

This document gives a high‑level overview of the main modules in the system
and how they relate to each other. The actual implementation is split across
separate repositories:

- `medagg-backend` – backend API (Django + DRF);
- `medagg-search` – search / query‑parsing logic;
- `medagg-frontend` – web UI (Vite + React + Ant Design).

---

## 1. Backend (`medagg-backend`)

### 1.1 Responsibilities

- expose REST API for:
  - authentication and user management;
  - dataset catalog (list, filter, detail);
  - search integration;
  - collection management (create, list, download);
  - admin operations;
- store and validate data in PostgreSQL;
- coordinate background work for building collections (if used).

### 1.2 Main components (conceptual)

- Django models for datasets, images, vocabularies, collections, etc.;
- DRF serializers and viewsets for API endpoints;
- authentication and permission configuration;
- settings for database, storage paths, CORS, etc.

The backend may also integrate with the `medagg-search` module – either
via a Python import or a service call – to interpret free‑text queries.

Relevant documentation:

- `api-overview.md`
- `db-schema.md`
- ADRs related to backend technology choices.

---

## 2. Search module (`medagg-search`)

### 2.1 Responsibilities

- parse free‑text user queries in Russian / English;
- map terms to elements of a **taxonomy** (stored in YAML or similar);
- extract structured slots:
  - modality;
  - anatomical area;
  - pathology / keyword;
  - task type;
- optionally compute relevance scores for datasets.

### 2.2 Typical inputs / outputs

Input from backend:

- query text as entered by user;
- optional context (language, default area, etc.).

Output to backend:

- normalized query representation;
- list of candidate datasets or filter suggestions;
- scores or ranking hints.

The exact integration contract should be refined based on the current
`medagg-search` implementation.

---

## 3. Frontend (`medagg-frontend`)

### 3.1 Responsibilities

- provide the main user interface for search, filtering and collection creation;
- authenticate users and store tokens / session state on the client;
- call backend API and display results;
- provide admin UI for dataset and vocabulary management (where applicable).

### 3.2 Stack

- Vite (build tooling, dev server);
- React (component model);
- Ant Design (UI components and layout).

Important screens described in:

- `user-guide.md`
- `admin-guide.md`

Screenshots for these sections should be stored under `docs/screenshots/`
(or similar) and referenced via placeholders like `<screenshot: ...>`.

---

## 4. Auxiliary modules / services

Depending on the final implementation, additional modules may include:

- background workers (Celery, RQ, Django Q, etc.) for long‑running tasks
  such as building large collections;
- storage adapters:
  - local filesystem for MVP;
  - later – cloud storage (S3‑compatible, etc.);
- monitoring and logging integrations.

These elements are not strictly required for the MVP, but should be kept in mind
when evolving the architecture.
