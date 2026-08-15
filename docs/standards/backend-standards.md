# Backend Standards (Python / FastAPI)

## Tooling

- **Lint/format:** `ruff` (replaces black + isort + flake8) — `ruff check . && ruff format .`.
- **Types:** `mypy` (strict on new code).
- **Python:** 3.12; type hints everywhere.

## Structure & layering

- Routers → services → repositories/ORM. Routers never write raw SQL.
- Validation lives in Pydantic schemas + a service layer; never only in the UI.
- Storage access goes through `StorageBackend` only.
- Settings via `pydantic-settings`, read from env — no hardcoded config.

## API conventions

- Return Pydantic DTOs (not ORM objects) from routes.
- Uniform error envelope: `{"error": {"code", "message", "details?"}}` (see [`api/errors.md`](../api/errors.md)).
- IDs are UUIDs; timestamps ISO-8601 UTC.
- Use `async def` for I/O-bound routes with async SQLAlchemy.

## Security

- Roles enforced via FastAPI dependencies (`require_admin`) at registration.
- Passwords hashed (argon2/bcrypt); JWTs short-lived.
- Never log secrets, tokens, or full credentials.

## Style summary

- `snake_case` functions/vars; `PascalCase` classes; `UPPER_CASE` constants.
- Meaningful names over clever ones; one public class per module where sensible.
- Docstrings for services and public modules; keep comments for "why", not "what".
