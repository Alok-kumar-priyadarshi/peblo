# ADR-003 — Single storage abstraction

- **Status:** accepted

## Context

Artwork and catalogues must be stored. Production uses Cloudflare R2; local dev/MinIO is acceptable.
The challenge requires that swapping backends be "one class."

## Decision

Define a `StorageBackend` ABC (`put`, `get`, `delete`, `exists`, `put_atomic`, `url`) with
`LocalDiskStorage`, `MinioStorage`, and `CloudflareR2Storage` implementations, selected by
`STORAGE_BACKEND` env var.

## Consequences

- **Positive:** no business code knows where bytes live; R2 swap = one new class + env vars.
- **Positive:** tests can use an in-memory fake.
- **Cost:** the interface must cover atomic write semantics for every backend (see ADR-002).
