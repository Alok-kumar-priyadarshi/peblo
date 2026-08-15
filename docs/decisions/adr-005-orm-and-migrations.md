# ADR-005 — SQLAlchemy 2.0 + Alembic

- **Status:** accepted

## Context

Need a clean schema with migrations; the publish job needs explicit transaction control.

## Decision

SQLAlchemy 2.0 (async) for models/ORM, Alembic for versioned migrations, Pydantic v2 for validation
and DTOs.

## Consequences

- **Positive:** mature migration tooling; explicit transactions for atomic publish.
- **Positive:** enforces invariants at the DB layer (constraints) with app-layer checks for
  cross-table rules.
- **Cost:** async SQLAlchemy adds some complexity vs sync; worth it for concurrent I/O under uvicorn.
