# ADR-0003 – Database Schema and Core Relations (MVP)

- Status: accepted
- Date: 2025-11-17
- Owners: DBAs, Backend Dev
- Related docs:
  - `docs/db-schema.md`
  - `docs/db-schema.ru.md`
  - `docs/diagrams/er-medagg-core.png`
  - SQL script from DBAs: `БД.sql`

## Context

The system needs to represent:

- **source datasets** (external collections of images);
- **reference dictionaries** describing those datasets
  (anatomical areas, modalities, ML task types, tags);
- **user activity** related to search and downloaded collections.

In the MVP, the team decided **not** to model individual images as rows in
the database. Instead, the DB tracks datasets and user-formed collections
of datasets, while image-level details stay in the original sources or the
filesystem structure.

The schema must:

- be simple enough for the MVP and demo;
- support efficient search / filtering in the catalog;
- allow building and expiring user collections.

## Decision

We chose a **relational schema in PostgreSQL** with the following core entities:

- `roles`, `users` – user accounts and roles;
- `anatomical_areas`, `modalities`, `ml_tasks`, `tags` – reference dictionaries;
- `datasets` – source datasets with basic metadata and links to storage;
- `dataset_modalities`, `dataset_ml_tasks`, `dataset_tags` – M:N links;
- `user_search_queries` – history of search queries and filter snapshots;
- `user_dataset_collections` – collections of datasets built for a user,
  including storage path and TTL;
- `collection_datasets` – link table between collections and datasets
  with optional relevance scores.

This model is documented in detail in **`docs/db-schema.md`** (and
`docs/db-schema.ru.md`) and is visualized in `docs/diagrams/er-medagg-core.png`.

## Alternatives considered

### A. Per-image modeling in the DB

Add tables like `images` and `image_labels` with one row per image.

- Pros:
  - fine-grained tracking of every image included in a user collection;
  - flexible analytics on labels and image usage.
- Cons:
  - more complex schema and migrations;
  - significantly larger tables;
  - not strictly required for the current MVP goals.

### B. Store everything as JSON blobs

Keep most dataset metadata in JSON fields instead of normalized tables.

- Pros:
  - very flexible schema, easier to evolve;
- Cons:
  - harder to query efficiently;
  - less transparent for DBAs and analytical queries;
  - still need some structure for vocabularies and relations.

## Consequences

Positive:

- clear, explicit data model for datasets and collections;
- DBAs can reason about indices, constraints and performance;
- user collections are reproducible via `user_dataset_collections`
  and `collection_datasets`;
- the schema matches the SQL script maintained by the DBAs.

Negative / risks:

- image-level statistics and analytics must be computed from external sources
  or from filesystem-level metadata, not directly from the relational schema;
- if future versions require strict tracking of individual images, the schema
  will need to be extended (e.g. adding `images` / `image_labels`).

## Notes

- The field `storage_path_hdfs` in `user_dataset_collections` is generic in
  meaning: it can point to local disk, a mounted volume, HDFS, object storage,
  etc. The exact infrastructure is defined by DevOps / deployment decisions.
- An expiration trigger sets `expires_at = created_at + 1 day` for user
  collections; background jobs or cron scripts are expected to remove
  expired archives from storage.

## References

- `docs/db-schema.md`
- `docs/db-schema.ru.md`
- ER diagram: `docs/diagrams/er-medagg-core.png`
- SQL script: `БД.sql`
