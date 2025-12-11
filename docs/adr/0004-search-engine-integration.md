# ADR-0004 – Search Engine and medagg-search Integration

- Status: accepted
- Date: 2025-11-24
- Owners: Backend Dev, Search/ML Engineer
- Related repos: `medagg-search`

## Context

Users need to search the dataset catalog using both:

- free-text queries (RU/EN), e.g. “pneumonia chest xray”;
- structured filters (modality, area, task type, tags).

The team planned to implement a separate search / query parsing module
(`medagg-search`) that:

- uses a **YAML taxonomy** for medical and technical terms;
- parses free-text into structured slots;
- can compute relevance scores for datasets.

We need to define how this module integrates with the Django backend.

## Decision

We decided to:

1. Keep `medagg-search` as a **separate module / library** (not a monolith).
2. Integrate it into the backend by:
   - calling it from search-related views in `medagg-backend`;
   - passing user query text and optional context;
   - receiving normalized query structure and ranking hints.
3. Let PostgreSQL remain the primary storage and source of truth;
   the search module **does not replace** the relational catalog.

## Alternatives considered

- **Full-text search directly in PostgreSQL** only
  - Pros:
    - no extra module;
    - simpler deployment.
  - Cons:
    - harder to implement domain-specific taxonomies and RU/EN normalization;
    - more limited control over query interpretation.

- **External search engine** (Elasticsearch, OpenSearch, etc.)
  - Pros:
    - powerful search capabilities;
  - Cons:
    - additional infrastructure complexity;
    - potentially overkill for the MVP.

## Consequences

Positive:

- clear separation between catalog storage (PostgreSQL) and query logic
  (`medagg-search`);
- easier to iterate on search behavior without touching DB schema;
- RU/EN and medical vocabulary logic stays close to the search code.

Negative / risks:

- integration and deployment of an extra Python module;
- need to keep taxonomy files and backend expectations in sync.

## References

- `medagg-search` repository
- search-related endpoints in `medagg-backend`.
