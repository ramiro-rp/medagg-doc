# MedAgg Documentation Repository

This repository contains **product and high‑level technical documentation**
for the *Medical Dataset Aggregator* project.

It is a **documentation umbrella**: the source code and low‑level technical
details live in separate repositories (backend, frontend, search tooling).
This repo focuses on **what the system does**, **how to deploy the MVP**,
and the **API/DB contracts** needed by the team.

## Repositories (reference)

- **Backend API (Django + DRF)** – `medagg-backend`  
  GitHub (reference): <https://github.com/b-barsky/medagg-backend>
- **Frontend (Vite + React + Ant Design)** – `frontend-DataHive`  
  GitHub: <https://github.com/Nikitmen/frontend-DataHive>
- **Search engine module** – `medagg-search`  
  (in the current MVP backend snapshot, the search code is also vendored under
  `src/libs/medsearch/`)


> If your team uses forks/branches (e.g. `feature/readme` / `feature/search`),
> treat those as the **MVP reference** when validating docs.


## Documentation structure

> Note: **User/Admin guides** are planned for the next iteration and are not published in the current doc set.

- `docs/api-overview.md` / `.ru.md` – API overview (MVP contract)
- `docs/deploy-guide.md` / `.ru.md` – deployment guide (single-node MVP)
- `docs/db-schema.md` / `.ru.md` – MVP database entities/relations
- `docs/modules.md` – system modules overview (with links to repos)
- `docs/adr/` – Architecture Decision Records (ADRs)
- `docs/diagrams/` – ER / architecture diagrams
- `docs/screenshots/` – screenshots referenced from the guides
- `docs/review-checklist.md` – who should review what


> If your team uses forks/branches (e.g. `feature/readme` / `feature/search`),
> treat those as the **MVP reference** when validating docs.
