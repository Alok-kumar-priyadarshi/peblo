# Database

| Document | Contents |
|---|---|
| [`er-diagram.md`](./er-diagram.md) | Mermaid `erDiagram` of the full schema |
| [`schema.md`](./schema.md) | Complete DDL (PostgreSQL 16) |
| [`indexes.md`](./indexes.md) | Indexes and the queries they serve |
| [`migrations.md`](./migrations.md) | Alembic migration strategy |

**Conventions:** UUID primary keys, `TIMESTAMPTZ`, enums for constrained vocabularies, FK
`ON DELETE CASCADE` for ownership chains, and `updated_at` triggers on mutable tables.
