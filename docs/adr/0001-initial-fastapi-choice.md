# ADR-0001 – Initial Choice of FastAPI for the Backend

- Status: superseded by ADR-0002
- Date: 2025-10-06
- Owners: Backend Dev, Team Lead
- Related meetings: early October 2025 Monday meetings

## Context

At the start of the project, the team needed to choose a backend
framework for the MVP of **Medical Imaging Dataset Aggregator**.

Requirements included:

- quick development of a REST API;
- good support for OpenAPI schema and interactive docs;
- comfortable developer experience with Python;
- reasonable performance for IO-bound workloads.

FastAPI was initially attractive because:

- it has first-class OpenAPI support and automatic docs (Swagger / ReDoc);
- it is async-friendly by design;
- it has a modern, type-hinted API style.

## Decision

We decided to start the backend implementation with **FastAPI**.

## Alternatives considered

- **Django + Django REST Framework**
  - Pros:
    - batteries-included framework (auth, admin, ORM, forms);
    - widely used and well understood by many developers;
    - rich ecosystem and community;
  - Cons (at that moment):
    - heavier to bootstrap;
    - more boilerplate for a small MVP.

- **Flask + extensions**
  - Pros:
    - lightweight and flexible;
  - Cons:
    - more manual work for auth, schema, validation, etc.;
    - not a clear advantage over FastAPI for this use case.

## Consequences

Positive:

- we could quickly prototype REST endpoints and interactive docs;
- the team explored FastAPI’s strengths and limitations for this domain.

Negative / risk:

- later, the team realized that they needed more of a “full framework”
  (especially for admin interfaces and ORM integration);
- this led to a follow-up decision (ADR-0002) to migrate to Django + DRF.

## References

- ADR-0002 – Switch to Django + DRF
- Early backend prototypes using FastAPI.
