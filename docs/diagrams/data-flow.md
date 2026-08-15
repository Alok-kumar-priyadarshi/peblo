# Data Flow

## Write path (CMS → storage)

```mermaid
flowchart LR
    E["Editor"] -->|"CRUD"| CMS["CMS"]
    CMS -->|"REST /admin/*"| API["FastAPI"]
    API -->|"SQL"| PG[("PostgreSQL")]
    API -->|"put artwork"| ST[("Storage")]
    subgraph Publish["Publish (admin)"]
        P1["CatalogueBuilder<br/>(read published, collapse groups)"]
        P2["AtomicWriter<br/>(tmp → rename)"]
    end
    API --> Publish
    Publish --> ST
    PG --> Publish
```

## Read path (storage → viewer)

```mermaid
flowchart LR
    V["Viewer (child)"] -->|"GET /catalog, /catalog/search"| API["FastAPI"]
    API -->|"get catalogue.json"| ST[("Storage")]
    ST --> API
    API --> V
```

## Materialised data lineage

```mermaid
flowchart TB
    SEED["seed_shows.json<br/>(95 rows, imperfect)"] -->|"seed job (upsert, quarantine)"| PG[("PostgreSQL")]
    REF["reference.json<br/>(sections, categories, langs, specs)"] -->|"validation config"| API["FastAPI"]
    API -->|"validates against REF"| PG
    PG -->|"publish job"| CAT["catalogue.json<br/>(collapsed, grouped, ordered)"]
    CAT -->|"served read-only"| VIEWER["Viewer UI"]
```

## Why the read path is decoupled

- The viewer never hits `admin` endpoints and never queries the DB directly.
- Publishing is a **materialisation** step: the DB is the source of truth for the CMS; the catalogue
  file is the source of truth for the viewer. This keeps the read path fast, cacheable, and isolated
  from CMS load and schema churn.
