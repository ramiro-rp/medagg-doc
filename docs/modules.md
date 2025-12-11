# System Modules

This document gives a high‑level overview of the main modules in the system
and how they relate to each other. The actual implementation is split across
separate repositories; for **low‑level technical details, installation and
developer instructions**, always refer to the README of the corresponding repo.

---

## 0. Repositories

- **Backend API** – `medagg-backend`  
  GitHub: <https://github.com/b-barsky/medagg-backend>  
  Main docs: `README.md` in that repo (API endpoints, settings, docker-compose).

- **Search module** – `medagg-search`  
  GitHub: <https://github.com/ESBehtev/medagg-search>  
  Main docs: `README.md` in that repo (taxonomy format, query parsing, examples).

- **Frontend** – `medagg-frontend`  
  GitHub: <https://github.com/solinoid/medagg-frontend>  
  Main docs: `README.md` in that repo (Vite dev server, build, environment vars).

This `medagg-docs` repository does **not** duplicate all technical details –
instead it links to the module‑specific READMEs where appropriate.

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

For **concrete endpoints, settings and docker-compose usage**, see:

- `medagg-backend` → [`README.md`](https://github.com/b-barsky/medagg-backend#readme)

Additional related docs in this repo:

- `docs/api-overview.md`
- `docs/db-schema.md`
- ADRs related to backend technology choices (`docs/adr/0001*`, `0002*`, `0003*`).

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

### 2.2 Integration contract

Input from backend:

- query text as entered by user;
- optional context (language, default area, etc.).

Output to backend:

- normalized query representation;
- list of candidate datasets or filter suggestions;
- scores or ranking hints.

For **taxonomy structure, configuration and usage examples**, see:

- `medagg-search` → [`README.md`](https://github.com/ESBehtev/medagg-search#readme)

Related ADR:

- `docs/adr/0004-search-engine-integration.md` – explains the integration strategy.

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

For **dev commands, build & deploy instructions and environment variables**, see:

- `medagg-frontend` → [`README.md`](https://github.com/solinoid/medagg-frontend#readme)

Screens and flows are described at a product level in:

- `docs/user-guide.md` / `.ru.md`
- `docs/admin-guide.md` / `.ru.md`

Corresponding screenshots should be placed in `docs/screenshots/` and linked
from those guides.

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
