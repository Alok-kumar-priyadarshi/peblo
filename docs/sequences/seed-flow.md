# Sequence — Seed / Migration

`docker-compose up` runs a one-shot seed job after migrations, loading `seed_shows.json`.

```mermaid
sequenceDiagram
    autonumber
    participant C as docker-compose
    participant Seed as seed job
    participant M as Alembic
    participant DB as PostgreSQL
    participant ST as StorageBackend

    C->>Seed: start (depends_on: api healthy)
    Seed->>M: alembic upgrade head
    M->>DB: apply migrations
    DB-->>M: schema ready

    Seed->>Seed: parse seed_shows.json
    loop each episode row
        Seed->>DB: upsert show / season / episode (keyed by slug, number, content_group+language)
        alt duplicate content_group+language
            Seed->>Seed: quarantine + report (do not double-insert)
        end
    end

    Seed->>ST: import provided assets/ → artwork objects
    ST-->>Seed: object keys
    Seed->>DB: INSERT artworks (only for files that actually exist)
    Seed-->>C: exit 0 (report: 95 imported, 1 duplicate quarantined, ...)
```

## Idempotency

The seed job is **idempotent**: upserts keyed on natural keys (slug; season number; content_group +
language), so re-running `docker-compose up` doesn't duplicate data. Duplicate `(content_group,
language)` rows are quarantined with a logged report rather than crashing the whole import.
