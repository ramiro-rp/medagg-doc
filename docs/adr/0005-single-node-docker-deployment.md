# ADR-0005 – Single-Node Deployment with Docker (MVP)

- Status: accepted
- Date: 2025-12-01
- Owners: Team Lead / DevOps
- Related docs: `docs/deploy-guide.md`, `docker-compose.yml`

## Context

For the MVP and demo phase, the application needs a **simple and reproducible**
deployment model that:

- works on a single host (laptop, small server, VPS, or Raspberry Pi 4);
- is easy for students / team members to run;
- does not require complex orchestration infrastructure.

The components are:

- Django + DRF backend;
- PostgreSQL;
- Vite/React frontend (static build);
- local file storage for ZIP archives.

## Decision

We decided to:

- standardize on **Docker + docker-compose** for the MVP deployment;
- run all components on a single Linux host;
- keep configuration in env files and compose file(s).

A future move to Kubernetes or cloud-managed services is out of scope
for the MVP, but not blocked by this decision.

## Alternatives considered

- **Bare-metal / manual deployment** without containers
  - Pros:
    - fewer layers in theory;
  - Cons:
    - harder to reproduce environments;
    - more manual setup for each machine.

- **Immediate Kubernetes deployment**
  - Pros:
    - closer to production-style cloud environments;
  - Cons:
    - too heavy for a student/demo project;
    - higher learning and operational overhead.

## Consequences

Positive:

- simple, scriptable startup using `docker compose up -d`;
- consistent environment between development and demo deployments;
- easy to document and teach.

Negative / risks:

- scaling and high availability are not addressed;
- performance on very small hardware (e.g. Raspberry Pi) may be limited
  and requires tuning (e.g. DB settings, swap, storage).

## References

- `docker-compose.yml` in the backend repository (or root deployment repo)
- `docs/deploy-guide.md`.
