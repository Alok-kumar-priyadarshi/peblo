# ADR-002 — Atomic publish

- **Status:** accepted

## Context

A reader must never observe a half-written catalogue. The anti-pattern the challenge flags is
"publishing by overwriting the live file."

## Decision

Write to a temporary key, then atomically rename (POSIX `os.replace`) for local/MinIO, or use a
conditional put / server-side copy for Cloudflare R2. Keep a stable live key
(`catalogue/catalogue.json`) and a versioned copy (`catalogue/catalogue-{version}.json`) for audit
and rollback.

## Consequences

- **Guarantee:** a read of the live key returns either the old or the new catalogue, never a partial
  one.
- **Crash safety:** if the process dies mid-publish, the live file is untouched; a watchdog reconciles
  the `running` run to `failed`.
- **Cost:** requires each storage backend to implement `put_atomic` semantics (trivial for local
  disk; a documented R2 path for production).
