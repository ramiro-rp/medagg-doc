# MedAgg Documentation Repository

This repository contains **product and high‑level technical documentation**
for the *Medical Dataset Aggregator* project.

It is a **documentation umbrella**: the source code and low‑level technical
details live in three main repositories:

- **Backend API** – `medagg-backend`  
  GitHub: <https://github.com/b-barsky/medagg-backend>

- **Search module** – `medagg-search`  
  GitHub: <https://github.com/ESBehtev/medagg-search>

- **Frontend** – `medagg-frontend`  
  GitHub: <https://github.com/solinoid/medagg-frontend>

For things like:

- concrete endpoints and serializers,
- exact DB migrations,
- npm / poetry / pip commands,
- local dev setup,

always refer to the **README.md** and docs inside those code repositories.

This repo focuses on:

- product‑level description;
- cross‑module architecture;
- ADRs and decisions;
- guides for end users, admins and deployment.

## Docs‑as‑Code

The documentation is maintained in a **docs‑as‑code** style:

- Markdown files in `docs/`
- Versioned together with the rest of the project
- Reviewed via pull requests

## Structure (proposed)

- `docs/product-overview.md` / `.ru.md` – high‑level product description
- `docs/user-guide.md` / `.ru.md` – end‑user guide
- `docs/admin-guide.md` / `.ru.md` – admin guide
- `docs/api-overview.md` / `.ru.md` – API overview (high‑level)
- `docs/deploy-guide.md` / `.ru.md` – deployment guide
- `docs/db-schema.md` – logical database schema
- `docs/modules.md` – system modules overview (with links to code repos)
- `docs/adr/` – Architecture Decision Records (ADRs)
- `docs/diagrams/` – ER / architecture diagrams
- `docs/screenshots/` – screenshots referenced from the guides
- `docs/review-checklist.md` – who should review what

See `docs/review-checklist.md` for which team role is expected to
validate each document.
=======
# medagg-doc
Product documentation for Medical Dataset Aggregator