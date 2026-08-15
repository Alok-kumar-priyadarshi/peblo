# Peblo TV Mini — Documentation Hub

Welcome to the engineering documentation for **Peblo TV Mini**, a miniature of Peblo's streaming product surface:

> A content team uploads a show's episodes and artwork through an **internal CMS**; the backend builds a
> **published catalogue file**; and a **viewer-facing UI** reads that catalogue so a child can browse,
> search, and filter — Netflix-style.

```
CMS (React) ──► API (FastAPI + Postgres) ──► publish job ──► catalogue.json in storage
                                                                │
                                         Viewer UI (React) ◄────┘
```

## How to use this documentation

All diagrams are authored in **[Mermaid](https://mermaid.js.org/)** and render natively on GitHub,
GitLab, and most Markdown viewers. Open any `.md` file to see them inline.

| Folder | What it contains |
|---|---|
| [`context/`](./context/) | Why this exists: project overview, domain model, glossary, seed-data analysis |
| [`architecture/`](./architecture/) | Architectural designs — C4 context/container/component, backend, frontend, storage, deployment |
| [`sequences/`](./sequences/) | Sequential designs — UML sequence diagrams for every key flow |
| [`database/`](./database/) | DB ER diagram (Mermaid), full schema (DDL), migrations, indexes |
| [`diagrams/`](./diagrams/) | Other diagrams — state machines, data-flow, activity, catalogue JSON shape |
| [`api/`](./api/) | API contracts — OpenAPI 3.1 spec, endpoint reference, errors, auth model |
| [`tasks/`](./tasks/) | Task breakdown (Parts A–E), backlog, milestones, acceptance criteria |
| [`instructions/`](./instructions/) | How to run, contribute, review, and use AI tools |
| [`standards/`](./standards/) | Coding standards — backend, frontend, Git, testing, database |
| [`decisions/`](./decisions/) | Architecture Decision Records (ADRs) — *recommended folder* |
| [`assets/`](./assets/) | Rendered images / static assets — *recommended folder* |

## Reading order (suggested)

1. [`context/project-overview.md`](./context/project-overview.md) — the 30-second orientation
2. [`context/domain-model.md`](./context/domain-model.md) — the vocabulary and invariants
3. [`architecture/system-architecture.md`](./architecture/system-architecture.md) — the big picture
4. [`database/schema.md`](./database/schema.md) + [`database/er-diagram.md`](./database/er-diagram.md)
5. [`sequences/`](./sequences/) — walk the key flows
6. [`api/api-contract.md`](./api/api-contract.md) — the contract the three layers agree on
7. [`tasks/`](./tasks/) — how the work breaks down
8. [`standards/`](./standards/) — the rules of the road before you write code

## Conventions used everywhere

- **Season 0** is reserved for trailers and is *not* a normal season in the viewer UI.
- **`content_group`** — episodes sharing one are language variants of the *same* episode; they must
  collapse into **one catalogue entry** listing its available languages.
- Roles: **`editor`** (CRUD) vs **`admin`** (CRUD + publish) — enforced, not just declared.
