# ADR-004 — JWT with role claim

- **Status:** accepted

## Context

Two roles: `editor` (CRUD) and `admin` (CRUD + publish), and they must be "actually enforced, not
just declared."

## Decision

Stateless JWT (Bearer) signed with a server secret; claims `sub`, `role`, `exp`. Roles enforced by
FastAPI dependencies (`get_current_user`, `require_admin`) attached at route registration.

## Consequences

- **Positive:** enforcement happens before handlers run — no `if role` sprinkled through code.
- **Positive:** stateless; no server-side session store.
- **Trade-off:** role changes require a new token (acceptable; short expiry mitigates).
- **Requirement:** `SECRET_KEY` must be managed as a secret.
