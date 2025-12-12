# ADR-0006 – Database Engine Choice: PostgreSQL

- Status: accepted
- Date: 2025-10-20
- Owners: DBAs, Backend Dev, Team Lead
- Related docs:
  - `docs/db-schema.md`
  - `docs/db-schema.ru.md`
  - `docs/adr/0003-database-schema-and-relations.md`

## Context

At the beginning of the project, the team needed to choose a **relational
database engine** for the MedAgg MVP. Requirements:

- reliable support for transactional workloads;
- good integration with Django ORM;
- open-source and widely available for students;
- strong JSON / full-text features as a bonus, but not mandatory.

The DBAs also preferred a system with which they already had production
experience.

## Decision

We decided to use **PostgreSQL** as the primary database engine for the MVP.

## Rationale

Main reasons for choosing PostgreSQL:

- first-class support in Django ORM and the Python ecosystem;
- powerful SQL dialect with JSONB, window functions and indexing options;
- open-source, widely used in both industry and academia;
- easy to run in Docker, including on resource-constrained machines
  (e.g. a small server or Raspberry Pi);
- the DBAs on the project are already familiar with PostgreSQL tooling
  and tuning.

## Alternatives considered

- **MySQL / MariaDB**  
  Pros:
  - also widely used, good tooling;  
  Cons:
  - slightly weaker integration with some Django features the team wanted,
    and the DBAs are more comfortable with PostgreSQL.

- **SQLite** (for everything, not just dev)  
  Pros:
  - extremely simple to set up, file-based;  
  Cons:
  - not ideal for concurrent access and multi-user server deployments;
  - harder to manage in a containerized environment for this use case.

## Consequences

Positive:

- consistent choice of database across dev and demo environments;
- ability to use PostgreSQL-specific features later if needed
  (JSONB, full-text search, etc.);
- aligns with existing DBA skills and existing `.sql` schema.

Negative / risks:

- requires running a dedicated DB service (container) even in small demos;
- slightly more operational overhead than SQLite-only setups, but acceptable
  for the MVP.

## References

- Docker configuration for the DB service in `docker-compose.yml`
- `БД.sql` – actual SQL schema for PostgreSQL.
