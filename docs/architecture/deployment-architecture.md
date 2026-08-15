# Deployment & Pipeline Architecture

## docker-compose topology

```mermaid
flowchart TB
    subgraph Compose["docker-compose.yml"]
        LB["api<br/>(FastAPI, uvicorn)"]
        PG[("postgres:16")]
        CMS["cms<br/>(Vite dev server / nginx)"]
        VIEWER["viewer<br/>(Vite dev server / nginx)"]
        MINIO["minio<br/>(S3-compatible, optional)"]
        SEED["seed-job<br/>(one-shot: load seed_shows.json)"]
    end
    LB --> PG
    LB --> MINIO
    SEED --> PG
    CMS -->|"/admin/*"| LB
    VIEWER -->|"/catalog*, /health"| LB
```

| Service | Port | Purpose |
|---|---|---|
| `api` | 8000 | FastAPI app |
| `postgres` | 5432 | Database |
| `cms` | 5173 | Internal CMS |
| `viewer` | 5174 | Browse UI |
| `minio` | 9000 | Optional S3-compatible storage (else local volume) |
| `seed` | — | One-shot migration + seed loader, exits after success |

## GitHub Actions pipeline

```mermaid
flowchart LR
    push["push / PR"] --> lint["lint<br/>(ruff, eslint, tsc)"]
    lint --> test["test<br/>(pytest, vitest)"]
    test --> build["build images<br/>(backend, cms, viewer)"]
    build --> deploy["deploy (explained)"]
    deploy --> smoke["smoke / health check"]
```

| Job | What | Tooling |
|---|---|---|
| lint | Static checks on backend + both UIs | ruff, mypy (opt), eslint, tsc --noEmit |
| test | Unit + integration tests | pytest (with a Postgres service container), vitest + RTL |
| build | Build/push Docker images | docker buildx, GHCR |
| deploy | Explained step (no real cloud required) | See below |

**Deploy step (written + explained, not executed):** publish images to GHCR, then a CD step (e.g.
`kubectl apply` or a PaaS webhook) would roll the new images. The step is gated on `main` and runs
only after build succeeds. In a real environment this would target a staging cluster with a
blue/green or rolling update, and the health endpoint would gate traffic promotion.

## Health & alerting

- **`GET /health`** returns `{"status":"ok","db":"ok","storage":"ok"}` (or a 503 with the failing
  component).
- **One thing to alert on:** `publish_runs.status = 'failed'` (or a publish run that stays `running`
  beyond a threshold). Reasoning: a failed/stalled publish means the viewer is silently serving stale
  content — the highest-signal, lowest-noise operational failure for this product. Alert on that with
  an SLO of "no failed publish in the last 24h", not on CPU/latency.

## Environment variables (`.env.example` covers all)

```dotenv
# API
DATABASE_URL=postgresql+asyncpg://peblo:peblo@postgres:5432/peblo
SECRET_KEY=change-me
ACCESS_TOKEN_EXPIRE_MINUTES=60

# Storage backend: local | minio | r2
STORAGE_BACKEND=local
LOCAL_STORAGE_DIR=/data/storage

# MinIO (optional)
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=peblo
MINIO_SECURE=false

# Cloudflare R2 (production)
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_ACCOUNT_ID=
R2_BUCKET=

# CORS / UI origins
CORS_ORIGINS=http://localhost:5173,http://localhost:5174
```

**Secrets management (production):** never commit secrets; inject via a secret manager (AWS Secrets
Manager / GCP Secret Manager / HashiCorp Vault) or the platform's sealed secrets, referenced by the
deployment as env vars at runtime. `SECRET_KEY`, DB credentials, and R2 keys live there; `.env` is
local-only and git-ignored.
