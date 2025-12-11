# Review Checklist by Role

This document maps documentation files to **team roles** that should
review and validate them before the MVP is considered complete.

Roles in this project:

- **Team Lead / DevOps**
- **Backend Developer**
- **Frontend Developer**
- **DBA (Database Administrator)** – 2 people
- **Documentation Owner**

---

## 1. Core product docs

- `docs/product-overview.md` / `.ru.md`
  - **Must review:**
    - Team Lead / DevOps (overall scope & MVP boundaries)
    - Backend Dev (technical feasibility)
    - Frontend Dev (UI / UX aspects that are assumed)
  - **Nice to review:**
    - DBA (data-related implications)

- `docs/modules.md`
  - **Must review:**
    - Team Lead / DevOps
    - Backend Dev
  - **Nice to review:**
    - Frontend Dev
    - DBA

- `docs/db-schema.md`
  - **Must review:**
    - Both DBAs
    - Backend Dev
  - **Nice to review:**
    - Team Lead / DevOps

---

## 2. Guides

- `docs/user-guide.md` / `.ru.md`
  - **Must review:**
    - Frontend Dev (UI flow matches reality)
    - Documentation Owner
  - **Nice to review:**
    - Team Lead / DevOps

- `docs/admin-guide.md` / `.ru.md`
  - **Must review:**
    - Backend Dev (admin API and data model assumptions)
    - Team Lead / DevOps
  - **Nice to review:**
    - DBAs (admin workflows related to catalog and vocabularies)

- `docs/api-overview.md` / `.ru.md`
  - **Must review:**
    - Backend Dev
    - Team Lead / DevOps
  - **Nice to review:**
    - DBAs (read-side patterns, impact on queries)

- `docs/deploy-guide.md` / `.ru.md`
  - **Must review:**
    - Team Lead / DevOps
  - **Nice to review:**
    - Backend Dev (env & settings)
    - DBAs (DB-specific parts)

---

## 3. Architecture Decision Records (ADRs)

- `docs/adr/0000-adr-template.md`
- `docs/adr/0001-initial-fastapi-choice.md`
- `docs/adr/0002-switch-to-django-drf.md`
- `docs/adr/0003-database-schema-and-relations.md`
- `docs/adr/0004-search-engine-integration.md`
- `docs/adr/0005-single-node-docker-deployment.md`

  - **Must review:**
    - Team Lead / DevOps
    - Backend Dev
    - DBAs (for DB-related ADRs)
  - **Nice to review:**
    - Frontend Dev (for decisions affecting the API shape)

---

## 4. Visuals and assets

- `docs/diagrams/` (ER, architecture diagrams)
  - **Must review:**
    - Team Lead / DevOps
    - DBAs (for DB diagrams)
  - **Nice to review:**
    - Backend Dev
    - Frontend Dev

- `docs/screenshots/`
  - **Must review:**
    - Frontend Dev
    - Documentation Owner

---

## 5. Process

Before the MVP demo:

1. Each role goes through the files marked “Must review”.
2. Comments are provided via PRs or inline notes.
3. Documentation Owner updates both EN and RU versions where applicable.
