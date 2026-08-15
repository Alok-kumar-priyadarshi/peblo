# ADR-006 — Search strategy

- **Status:** accepted

## Context

`GET /catalog/search?q=&category=&language=&section=` with `q` matching show title, episode title,
and category; filters must compose. Seed data is ~95 episodes (8 shows).

## Decision

v1: SQL `ILIKE` over `shows.title`, `episodes.title`, and category, with filters as SQL `AND`.
Document the ceiling and the next step.

## Consequences

- **Positive:** simple, correct, sufficient for the seed and well into thousands of rows.
- **Cost:** `ILIKE '%…%'` can't use a B-tree index; degrades linearly at scale (tens of thousands of
  rows / sustained load).
- **Next step (when needed):** PostgreSQL `tsvector` + GIN index for `q`; `pg_trgm` for substring;
  keep filters on ordinary indexed columns. Optionally offload to a dedicated search engine later.
