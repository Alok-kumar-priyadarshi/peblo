# Architecture Decision Records (ADRs)

Lightweight records of significant, hard-to-reverse decisions. Each ADR is short and states context,
decision, and consequences.

| ADR | Decision |
|---|---|
| [`adr-001-serve-catalogue-file.md`](./adr-001-serve-catalogue-file.md) | Serve a pre-published catalogue file rather than querying the DB per request |
| [`adr-002-atomic-publish.md`](./adr-002-atomic-publish.md) | Atomic publish via temp-file + rename / conditional put |
| [`adr-003-storage-abstraction.md`](./adr-003-storage-abstraction.md) | Single `StorageBackend` interface for local/MinIO/R2 |
| [`adr-004-jwt-roles.md`](./adr-004-jwt-roles.md) | Stateless JWT with a role claim for authN/authZ |
| [`adr-005-orm-and-migrations.md`](./adr-005-orm-and-migrations.md) | SQLAlchemy 2.0 + Alembic |
| [`adr-006-search-strategy.md`](./adr-006-search-strategy.md) | ILIKE now, tsvector later |

## Template

```
# ADR-NNN — <title>
- Status: proposed | accepted | superseded
- Context:   the forces in play
- Decision:  what we chose
- Consequences: what improves / what it costs
```
