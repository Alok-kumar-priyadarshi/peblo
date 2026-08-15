# API Contracts

The contract the three layers agree on. The machine-readable source of truth is
[`openapi.yaml`](./openapi.yaml) (OpenAPI 3.1); the markdown files are the human-readable reference.

| Document | Contents |
|---|---|
| [`api-contract.md`](./api-contract.md) | Overview + endpoint index + shared conventions |
| [`endpoints.md`](./endpoints.md) | Per-endpoint request/response detail |
| [`errors.md`](./errors.md) | Error model + all error codes |
| [`auth.md`](./auth.md) | Authentication + role model |
| [`openapi.yaml`](./openapi.yaml) | OpenAPI 3.1 spec (generate typed clients from this) |

## Conventions

- **Base path:** `/` (admin under `/admin`, public under `/catalog`).
- **Auth:** `Authorization: Bearer <JWT>` on all `/admin/*` routes.
- **Content type:** `application/json`; uploads use `multipart/form-data`.
- **Errors:** uniform envelope (see [`errors.md`](./errors.md)).
- **IDs:** UUIDs; **timestamps:** ISO-8601 UTC.
