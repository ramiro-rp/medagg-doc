# System Modules

This document gives a high-level overview of the main modules in the system.
For module-specific setup and detailed technical usage, refer to each repo’s `README.md`.

---

## Repositories

- **Backend API** (`medagg-backend`)
  - GitHub: <https://github.com/b-barsky/medagg-backend>

- **Search module** (`medagg-search`)
  - GitHub: <https://github.com/ESBehtev/medagg-search>

- **Frontend** (`frontend-DataHive`)
  - GitHub: <https://github.com/Nikitmen/frontend-DataHive>

---

## Backend (`medagg-backend`)

- Django + DRF API (base path: `/api/v1/`)
- Dataset catalog (list, filters, detail)
- Search endpoint that delegates to the search module

### Dataset README generation (feature branch)

There is a backend feature (shared in the team chat) that generates a dataset README text based on the dataset metadata.

- Backend branch referenced by the team: `feature/search`
- Quick verification workflow (as provided by the developer):
  - Ensure the `data/` folder is empty
  - Start with: `python configure.py --docker`
  - Open Django shell: `docker-compose exec medagg-app python manage.py shell`
  - Create a test dataset and verify that README generation ran

(Any required DB schema change for storing generated README content is tracked in the review checklist.)

---

## Search module (`medagg-search`)

- Query parsing / normalization
- Matching against the taxonomy
- Ranking / scoring helpers (if enabled by backend integration)

---

## Frontend (`frontend-DataHive`)

- UI for searching datasets and viewing results/details
- Calls the backend API configured via `VITE_API_URL`
