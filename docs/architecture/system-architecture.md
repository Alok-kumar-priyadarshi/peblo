# System Architecture

## C4 — Context (Level 1)

```mermaid
flowchart LR
    Editor["👤 Content Editor<br/>(internal user)"]
    Admin["👤 Admin<br/>(publish + CRUD)"]
    Viewer["🧒 Viewer<br/>(end user)"]

    subgraph PebloTV["Peblo TV Mini — system boundary"]
        CMS["CMS<br/>React + TS"]
        API["API<br/>FastAPI + Postgres"]
        Store["Storage<br/>local / MinIO / R2"]
        VUI["Viewer UI<br/>React + TS"]
    end

    Editor -->|"CRUD shows/episodes, upload artwork"| CMS
    Admin -->|"CRUD + publish"| CMS
    CMS -->|"REST /admin/*"| API
    API -->|"reads/writes rows"| PG[("PostgreSQL")]
    API -->|"put/get objects"| Store
    API -->|"atomic write catalogue.json"| Store
    VUI -->|"GET /catalog*, /health"| API
    Viewer --> VUI
```

**Actors**
- **Editor** — content team; can create/edit shows, seasons, episodes, upload artwork.
- **Admin** — editor + publish authority.
- **Viewer** — a child browsing shows; anonymous, read-only.

## C4 — Containers (Level 2)

```mermaid
flowchart TB
    subgraph CMS_App["cms — React SPA (Vite)"]
        direction TB
        C1["List / Search / Filter / Pagination"]
        C2["Create/Edit form + 3 artwork slots"]
        C3["Publish page (report + button + history)"]
    end

    subgraph Viewer_App["viewer — React SPA (Vite)"]
        V1["Hero + rows"]
        V2["Search + filters"]
        V3["Show detail"]
    end

    subgraph API_App["backend — FastAPI"]
        A1["Admin router<br/>(auth, CRUD, upload, publish)"]
        A2["Catalog router<br/>(catalogue, search)"]
        A3["Health router"]
        S["StorageBackend<br/>(LocalDisk | MinIO | CloudflareR2)"]
        P["Publish service<br/>(build + atomic write)"]
    end

    DB[("PostgreSQL 16")]

    CMS_App -->|"HTTPS /admin/*"| API_App
    Viewer_App -->|"HTTPS /catalog*, /health"| API_App
    A1 --> DB
    A2 --> S
    P --> DB
    P --> S
    A3 --> DB
```

## Technology stack

| Concern | Choice | Rationale |
|---|---|---|
| API | Python 3.12, FastAPI | Async, typed, auto OpenAPI; required by challenge |
| ORM | SQLAlchemy 2.0 + Alembic | Mature migrations; explicit transactions for atomic publish |
| Validation | Pydantic v2 | Shares types with FastAPI; centralises rules |
| Database | PostgreSQL 16 | Required; JSONB for publish counts, real FK constraints |
| Storage | `StorageBackend` ABC → `LocalDiskStorage`, `CloudflareR2Storage` (boto3) | One-class swap |
| CMS / Viewer | React 18 + TypeScript + Vite | Required |
| Server state | TanStack Query | Caching, retries, optimistic states |
| Auth | JWT (Bearer) with role claim | Stateless, easy to test/enforce |
| Container | Docker + docker-compose | Part D requirement |
| CI | GitHub Actions | Part D requirement |
| Tests | pytest (backend), Vitest + Testing Library (frontend) | Standard |

## Key design decisions (see [`decisions/`](../decisions/) for full ADRs)

1. **Serve a pre-published catalogue file** rather than querying the DB per viewer request
   → fast, cacheable, decouples read path from the CMS. Trade-off: eventual consistency between a
   publish and what viewers see (acceptable — publishes are explicit).
2. **Atomic publish** via temp-file + rename (local/MinIO) or object-storage conditional put (R2).
3. **Collapse `content_group` at publish time** — the DB stores one row per language variant; the
   catalogue materialises the `languages` list.
4. **Single `StorageBackend` interface** so dev (local disk) and prod (R2) are one class apart.
