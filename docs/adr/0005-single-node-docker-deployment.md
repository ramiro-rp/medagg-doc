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

More advanced deployment options (multi-node clusters, managed PaaS, etc.)
are explicitly **out of scope** for the current MVP.

## Alternatives considered

- **Bare-metal / manual deployment** without containers  
  Pros:
  - fewer layers in theory;  
  Cons:
  - harder to reproduce environments;
  - more manual setup for each machine.

- **Immediately adopting a more complex orchestration platform**  
  (for example a container cluster or cloud PaaS)  
  Pros:
  - closer to production-grade deployments;  
  Cons:
  - too heavy for a demo project;
  - higher learning and operational overhead for the team.

## Consequences

Positive:

- simple, scriptable startup using `docker compose up -d`;
- consistent environment between development and demo deployments;
- easy to document and teach.

Negative / risks:

- scaling and high availability are not addressed in the MVP;
- performance on very small hardware (e.g. Raspberry Pi) may be limited
  and requires tuning (DB settings, swap, storage).

## References

- `docker-compose.yml` in the backend repository (or root deployment repo)
- `docs/deploy-guide.md`.
