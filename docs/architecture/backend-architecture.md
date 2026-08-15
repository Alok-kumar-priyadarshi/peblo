# Backend Architecture

## Component diagram

```mermaid
flowchart TB
    subgraph FastAPI["backend/app"]
        subgraph API["api/"]
            RAdmin["admin router<br/>shows, seasons, episodes, artwork, publish"]
            RCatalog["catalog router<br/>GET /catalog, /catalog/search"]
            RHealth["health router"]
        end
        subgraph Schemas["schemas/ (Pydantic)"]
            SDTO["request/response DTOs"]
            SVal["validators<br/>(aspect, size, uniqueness)"]
        end
        subgraph Core["core/"]
            CFG["config (.env)"]
            SEC["security (JWT, roles)"]
            DEP["dependencies<br/>(get_current_user, require_admin)"]
        end
        subgraph Publish["publish/"]
            BUILDER["catalogue builder"]
            ATOMIC["atomic writer"]
        end
        subgraph Storage["storage/"]
            ABC["StorageBackend (ABC)"]
            LOCAL["LocalDiskStorage"]
            R2["CloudflareR2Storage"]
        end
        DB["db/ — SQLAlchemy models + session"]
    end

    PG[("PostgreSQL")]

    RAdmin --> SDTO
    RAdmin --> SEC
    RAdmin --> DB
    RAdmin --> Storage
    RAdmin --> BUILDER
    RCatalog --> Storage
    RCatalog --> DB
    RHealth --> DB
    BUILDER --> DB
    BUILDER --> ATOMIC
    ATOMIC --> Storage
    DB --> PG
    ABC -.-> LOCAL
    ABC -.-> R2
```

## Layering rules

1. **Routers** never touch SQL directly — they call services/repositories and return Pydantic DTOs.
2. **Validation** lives in Pydantic schemas + a dedicated service; it is *never* client-side-only.
3. **Storage** is accessed only through `StorageBackend`; no `open()`/`boto3` in routers.
4. **Roles** are enforced by FastAPI dependencies (`require_admin`) on every admin endpoint — not
   merely checked inside handlers.
5. **Transactions** are explicit: publish runs in one transaction to read + record, then writes the
   catalogue atomically *after* commit.

## Module responsibilities

| Module | Responsibility |
|---|---|
| `api/admin` | Auth, CRUD for shows/seasons/episodes, artwork upload, publish trigger, run history, validation report |
| `api/catalog` | `GET /catalog`, `GET /catalog/search` — read-only, no auth |
| `api/health` | Liveness/readiness probe for orchestration + alerting |
| `schemas/` | Pydantic v2 models; embed invariant validators (aspect ratio, size, enum membership) |
| `core/config` | Env-driven settings (pydantic-settings) |
| `core/security` | Password hashing (argon2/bcrypt), JWT sign/verify, role extraction |
| `core/dependencies` | `get_current_user`, `require_admin`, `get_db` |
| `publish/builder` | Query published content, collapse content_groups, group by section, deterministic sort |
| `publish/atomic` | Write to temp key → atomic rename / conditional put; return final object key |
| `storage/` | `StorageBackend` + implementations |
| `db/` | SQLAlchemy models, session factory, engine |

## Request lifecycle (admin write)

```mermaid
sequenceDiagram
    participant C as CMS
    participant R as Router
    participant D as Deps (auth/roles)
    participant S as Service
    participant DB as PostgreSQL
    C->>R: POST /admin/episodes (Bearer token)
    R->>D: require_admin / get_current_user
    D->>DB: load user, verify JWT
    alt invalid / wrong role
        D-->>R: 401 / 403
        R-->>C: error JSON
    else ok
        D-->>R: user
        R->>S: create_episode(dto)
        S->>S: validate invariants
        S->>DB: INSERT ... RETURNING
        DB-->>S: row
        S-->>R: DTO
        R-->>C: 201 + resource
    end
```
