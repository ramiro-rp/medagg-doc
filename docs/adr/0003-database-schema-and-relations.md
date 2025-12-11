# ADR-0003 – Database Schema and Core Relations

- Status: accepted
- Date: 2025-11-17
- Owners: DBAs, Backend Dev
- Related docs: `docs/db-schema.md`

## Context

The system needs to represent:

- source datasets (external collections of images);
- individual images or studies;
- vocabularies (anatomical areas, modalities, task types, tags);
- user-built collections of images for ML experiments.

The schema must:

- be simple enough for the MVP;
- support efficient search / filtering in the catalog;
- allow building reproducible collections.

## Decision

We chose a **relational schema** with the following core entities:

- `Dataset` – source dataset;
- `Image` (or `Study`) – individual item within a dataset;
- `AnatomicalArea`, `Modality`, `TaskType`, `Tag` – vocabularies;
- `Collection` – user-built custom dataset;
- `CollectionItem` – linking table between `Collection` and `Image`;
- `Label` / `ImageLabel` – optional label modeling for classification tasks.

The high-level relations are described in `docs/db-schema.md` and
reflected in the Django models.

## Alternatives considered

- **Model only datasets, not individual images** in the DB
  - Pros:
    - simpler schema;
  - Cons:
    - limited flexibility for building fine-grained collections;
    - harder to trace which exact images ended up in a collection.

- **Store everything as JSON blobs**
  - Pros:
    - very flexible;
  - Cons:
    - harder to query efficiently;
    - less transparent for DBAs and analysts.

## Consequences

Positive:

- clear, explicit data model suitable for SQL queries;
- DBAs can reason about indices, constraints and performance;
- collections are reproducible via `Collection` and `CollectionItem` tables.

Negative / risks:

- for very large image corpora, per-image modeling may require careful
  indexing and partitioning in the future;
- DICOM-level details are out of MVP scope and would require further modeling
  if needed later.

## References

- `docs/db-schema.md`
- DB migration scripts in `medagg-backend`.
