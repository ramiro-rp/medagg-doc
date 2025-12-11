# ADR-0002 – Switch to Django + Django REST Framework

- Status: accepted
- Date: 2025-11-10
- Owners: Backend Dev, Team Lead
- Related meetings: November 2025 Monday meetings (backend refocus)

## Context

After initial experiments with **FastAPI**, the team reassessed backend needs
for the MVP and beyond. Key observations:

- the project requires a fairly standard **relational data model**
  (datasets, images, vocabularies, collections);
- we need a solid **admin interface** to manage datasets, vocabularies and
  user-related operations without building everything manually;
- most of the team already has production experience with **Django**.

Using FastAPI plus custom components for ORM, admin, migrations and configuration
started to look heavier than using a well-integrated Django stack.

## Decision

We decided to **switch the backend implementation to Django + Django REST Framework (DRF)**.

## Rationale

Main arguments for the switch:

- Django provides a robust, battle-tested ORM and admin interface;
- DRF offers a mature toolkit for building REST APIs, including serializers,
  viewsets, browsable API, throttling, etc.;
- the team’s collective experience with Django / DRF is higher than with FastAPI;
- for a typical IO-bound web API with relational DB, Django + DRF is more than
  sufficient and allows the team to move faster and more safely.

Regarding API documentation:

- instead of relying on FastAPI’s built-in OpenAPI docs, we can use
  **DRF extensions** (e.g. `drf-spectacular`, `drf-yasg`) to generate
  OpenAPI schemas and interactive docs;
- this keeps API documentation as part of the Django / DRF ecosystem.

## Alternatives considered

- **Stay on FastAPI and invest more in tooling**
  - Pros:
    - keep the work already done;
    - modern async-first design.
  - Cons:
    - extra effort for admin tooling and standard flows;
    - diverges from the team’s main expertise.

- **Hybrid approach**: Django for admin, FastAPI for public API
  - Pros:
    - each framework used where it shines.
  - Cons:
    - increased operational complexity;
    - two frameworks, two sets of patterns and configs.

## Consequences

Positive:

- simpler, more coherent backend stack;
- easier onboarding for new developers familiar with Django;
- out-of-the-box admin interface for managing catalog and vocabularies.

Negative / risks:

- DRF is not async-first by default, but this is acceptable for the current
  MVP requirements.

## References

- ADR-0001 – Initial Choice of FastAPI
- `medagg-backend` repository (Django + DRF implementation).
