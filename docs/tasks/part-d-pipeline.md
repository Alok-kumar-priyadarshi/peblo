# Part D — Pipeline & Operability Tasks

## T-OPS-01 — docker-compose
- **Acceptance:** `docker-compose up` brings up API, Postgres, CMS, Viewer, and a seed job; system is
  seeded and working first try. Health probe used as the readiness gate.
- **Reference:** [`architecture/deployment-architecture.md`](../architecture/deployment-architecture.md).

## T-OPS-02 — GitHub Actions
- **Acceptance:** workflow runs lint, tests (with Postgres service), and image builds on push/PR. A
  deploy step is written and explained (no real cloud needed).
- **Reference:** [`architecture/deployment-architecture.md`](../architecture/deployment-architecture.md).

## T-OPS-03 — `.env.example` + secrets
- **Acceptance:** every variable documented; a paragraph explains production secrets management
  (secret manager / sealed secrets, never committed).
- **Reference:** [`architecture/deployment-architecture.md`](../architecture/deployment-architecture.md).

## T-OPS-04 — Health + alerting
- **Acceptance:** `GET /health` reports db + storage status; one alert is chosen and justified
  (publish failure/stall). Documented in README.
- **Reference:** [`architecture/deployment-architecture.md`](../architecture/deployment-architecture.md).
