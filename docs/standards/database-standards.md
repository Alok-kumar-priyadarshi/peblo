# Database Standards

## Modelling

- UUID primary keys; `TIMESTAMPTZ` for all timestamps.
- Constrained vocabularies: use Postgres enums when fixed, a CHECK on `TEXT` when the set may grow
  from `reference.json` (see `shows.section`).
- FKs use `ON DELETE CASCADE` along ownership chains (show → season → episode).
- Enforce uniqueness at the DB level, not just in code — e.g. `(content_group, language)`.

## Naming

- Tables plural snake_case (`publish_runs`); columns singular snake_case (`show_id`).
- FK columns named `<singular_table>_id`.
- Unique constraints named `uq_<table>_<cols>`; checks `chk_<table>_<rule>`.

## Migrations

- Alembic; one logical change per revision; forward-only in v1.
- Schema and data migrations in separate revisions.
- Never edit an applied migration — add a new one.

## Indexes

- Index columns used in filters/joins/order-by; document each index's purpose in
  [`database/indexes.md`](../database/indexes.md).
- Know when an index **won't** help (e.g. leading-wildcard `ILIKE`); plan the `tsvector` upgrade
  (see [`sequences/search-flow.md`](../sequences/search-flow.md)).

## Transactions

- Publish reads + run-record in one transaction; the catalogue write happens **after** commit and is
  atomic (temp + rename / conditional put).
