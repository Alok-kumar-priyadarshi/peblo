# Frontend Architecture

Two separate React apps — the **CMS** (internal, authenticated) and the **Viewer** (public,
read-only). They share no state and only meet at the published catalogue.

## CMS component diagram

```mermaid
flowchart TB
    subgraph CMS["cms/ (React + TS + TanStack Query)"]
        App["App + router"]
        Auth["Auth context + login"]
        subgraph Pages["pages"]
            List["Show/Episode list<br/>(search, filters, pagination)"]
            Edit["Create/Edit form"]
            Upload["ArtworkUpload slot ×3"]
            Publish["Publish page<br/>(report, button, history)"]
        end
        subgraph Data["data layer"]
            Queries["TanStack Query hooks"]
            ApiClient["typed API client"]
        end
        UI["UI primitives + states<br/>(loading, empty, error, 403)"]
    end
    BE["backend /admin/*"]

    App --> Auth
    List --> Queries
    Edit --> Upload
    Edit --> Queries
    Publish --> Queries
    Queries --> ApiClient
    ApiClient -->|"fetch + Bearer token"| BE
```

## Viewer component diagram

```mermaid
flowchart TB
    subgraph Viewer["viewer/ (React + TS + TanStack Query)"]
        VApp["App + router"]
        Home["Home: Hero + section rows"]
        Search["Search + filters"]
        Detail["Show detail"]
        Cards["Card primitives<br/>(banner/poster/thumbnail)"]
        VQueries["TanStack Query hooks"]
        VClient["typed API client"]
    end
    Cat["backend /catalog, /catalog/search"]

    VApp --> Home
    VApp --> Search
    VApp --> Detail
    Home --> Cards
    Detail --> Cards
    Home --> VQueries
    Search --> VQueries
    Detail --> VQueries
    VQueries --> VClient
    VClient --> Cat
```

## Artwork per surface (viewer rule)

| Surface | Artwork kind | Rationale |
|---|---|---|
| Featured hero | `banner` (16:9) | Wide cinematic hero |
| Section rows | `poster` (2:3) | Vertical cards in a row |
| Episode lists | `thumbnail` (16:9) | Small landscape tiles |

## State handling matrix (both apps)

| State | CMS behaviour | Viewer behaviour |
|---|---|---|
| Loading | Skeleton rows / spinners | Skeleton + lazy image with blur-up placeholder |
| Empty | "No shows match — clear filters" | Friendly empty state with clear CTA |
| Error | Inline, human-readable (editor can act) | Retry button + offline hint |
| Permission denied | 403 screen, hide admin-only actions | n/a (public) |
| Slow images | Preview spinner while uploading | Blur-up / background color placeholder, lazy loading |

## Shared conventions

- TanStack Query for server state; local component state only for transient UI.
- Typed API client generated from the OpenAPI contract (see [`api/`](../api/)).
- No business logic duplicated between CMS and Viewer; the API is the single source of truth.
