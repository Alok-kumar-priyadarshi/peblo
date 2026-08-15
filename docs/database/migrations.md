# Migrations

Tool: **Alembic** (SQLAlchemy 2.0). One migration per logical change; never edit an applied migration.

## Layout

```
backend/alembic/
├── alembic.ini
├── env.py
└── versions/
    ├── 0001_initial_schema.py     # enums + users + shows + seasons + episodes + artworks + publish_runs
    ├── 0002_seed_roles.py         # create default editor + admin users
    └── 0003_indexes.py            # optional performance indexes (from indexes.md)
```

## Conventions

1. **Forward-only** in v1 — no down migrations needed for a greenfield take-home; if you add `down`,
   keep it paired with `up`.
2. **Names** are `NNNN_snake_case.py` with a human summary.
3. **Enums** are created with `sa.Enum(..., name=...)` and `postgresql_using`/`create_type` so Alembic
   tracks them.
4. **Data migrations** (e.g. seeding roles) live in a separate revision from schema migrations.
5. Run with `alembic upgrade head` (the compose `seed` job does this automatically).

## Example revision header

```python
revision = "0001"
down_revision = None
branch_labels = None
depends_on = None
```

## Guardrails

- Migrations are applied **before** the seed job (see [`sequences/seed-flow.md`](../sequences/seed-flow.md)).
- CI runs `alembic upgrade head` against a scratch Postgres service to prove migrations are clean.
- The seed loader is **idempotent**, so `alembic upgrade head` + seed can be re-run safely.
