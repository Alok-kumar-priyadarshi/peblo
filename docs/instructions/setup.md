# Setup & Run

## Prerequisites

- Docker + docker-compose (recommended), or
- Python 3.12, PostgreSQL 16, Node 20, pnpm/npm (for local dev without Docker).

## Quick start (Docker)

```bash
cp .env.example .env          # review and adjust values
docker-compose up --build
```

This starts:

| Service | URL |
|---|---|
| API (FastAPI) | http://localhost:8000 |
| API docs (Swagger) | http://localhost:8000/docs |
| CMS | http://localhost:5173 |
| Viewer | http://localhost:5174 |
| PostgreSQL | localhost:5432 |
| MinIO (optional) | http://localhost:9001 |

The `seed` service runs `alembic upgrade head` and loads `seed_shows.json` once, then exits.

### Verify

```bash
curl http://localhost:8000/health
# {"status":"ok","db":"ok","storage":"ok"}

curl http://localhost:8000/catalog
# published catalogue JSON
```

### Seed login accounts

| Email | Role |
|---|---|
| `editor@peblo.tv` | editor |
| `admin@peblo.tv` | admin |

(Set real credentials via env vars in production — see `architecture/deployment-architecture.md`.)

## Local dev (no Docker)

```bash
# backend
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
alembic upgrade head
uvicorn app.main:app --reload

# cms
cd cms && npm install && npm run dev

# viewer
cd viewer && npm install && npm run dev
```

## Environment

Every variable is documented in `.env.example`. Secrets (DB password, `SECRET_KEY`, R2 keys) must
never be committed; inject them via a secret manager in production.
