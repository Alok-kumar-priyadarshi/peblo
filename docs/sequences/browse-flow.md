# Sequence — Viewer Catalogue Browse

The viewer talks **only** to read-only catalogue endpoints. No admin endpoints, no auth.

```mermaid
sequenceDiagram
    autonumber
    actor V as Viewer (child)
    participant UI as Viewer UI
    participant API as FastAPI
    participant ST as StorageBackend
    participant CDN as CDN/cache

    V->>UI: open app
    UI->>API: GET /catalog
    API->>CDN: cache lookup (ETag / max-age)
    alt cache hit
        CDN-->>API: cached catalogue
    else miss
        API->>ST: get("catalogue/catalogue.json")
        ST-->>API: catalogue JSON
        API-->>CDN: 200 + Cache-Control
    end
    API-->>UI: catalogue {sections: [...], ...}
    UI->>UI: render hero (banner) + rows (poster)

    V->>UI: tap a show
    UI->>UI: open show detail (already in catalogue)
    UI->>UI: list seasons (skip Season 0), episodes (thumbnail)
    UI->>UI: show grouped episode languages (en, hi)

    Note over UI,ST: Artwork images are lazy-loaded via signed/URLs from storage
```

## Design notes

- The catalogue is a **single document**; the viewer renders locally and never queries the DB.
- Artwork **URLs** are embedded in the catalogue; images load lazily with blur-up placeholders so the
  UI stays pleasant on slow networks.
- `Season 0` is filtered out client-side (and omitted from the catalogue's season list) so trailers
  never appear as a normal season.
