# Sequence — Publish (Atomic Catalogue Build)

The most important flow. Steps 1–2 record the run; 3–6 build; 7–8 atomically swap; 9 finalises.

```mermaid
sequenceDiagram
    autonumber
    actor A as Admin
    participant CMS as CMS
    participant API as FastAPI
    participant B as CatalogueBuilder
    participant DB as PostgreSQL
    participant ST as StorageBackend

    A->>CMS: click "Publish"
    CMS->>API: POST /admin/catalog/publish (Bearer admin)
    API->>API: require_admin
    API->>DB: INSERT publish_runs (status=running, triggered_by=admin) RETURNING id, version
    DB-->>API: run {id, version}

    API->>B: build(version)
    B->>DB: SELECT published shows + seasons + episodes + artworks
    DB-->>B: rows (draft/null-section excluded)
    B->>B: collapse content_group → entry with languages[]
    B->>B: group by section; deterministic order (slug, season, episode)
    B->>B: attach artwork urls per surface
    B-->>API: catalogue JSON

    API->>ST: put_atomic("catalogue/catalogue.json", json)
    Note over ST: write tmp → atomic rename / conditional put
    ST-->>API: ok
    API->>ST: put("catalogue/catalogue-{version}.json", json)  # versioned copy
    ST-->>API: ok

    API->>DB: UPDATE publish_runs SET status='succeeded', counts={...}, catalog_key=...
    DB-->>API: ok
    API-->>CMS: 200 {run_id, version, counts}
    CMS-->>A: "Published v42 — 7 shows, 90 episodes"

    Note over API,DB: If build fails → UPDATE publish_runs SET status='failed', error=...
```

## Failure semantics

- **Crash before the atomic write** → live catalogue untouched; run row left `running`/marked
  `failed` by a watchdog. Viewers still see the previous catalogue.
- **Crash during the write** → the atomic rename either happened or didn't; there is **no**
  half-written file. The run row is reconciled to `failed` on next health/publish.
- **Idempotency** — re-publishing with unchanged content yields an identical catalogue (deterministic
  ordering + stable keys), so a retried run is safe.
