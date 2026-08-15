# Project Overview

## What is Peblo TV Mini?

A miniature of Peblo's streaming mode ("Peblo TV"). It is a full-stack system with **three layers**
plus the **pipeline** that runs them:

| Layer | Tech | Audience | Reads/Writes |
|---|---|---|---|
| **CMS** | React + TypeScript | Internal content editors/admins | Writes |
| **API** | FastAPI + PostgreSQL | CMS + publish jobs | Writes + serves catalogue |
| **Viewer UI** | React + TypeScript | Children / end users | Reads published catalogue only |

## The core loop

1. An **editor** logs into the CMS and creates/edits shows, seasons, and episodes.
2. The editor uploads **artwork** (poster, banner, thumbnail) which is validated and stored behind a
   **storage abstraction**.
3. An **admin** triggers **publish**; the API builds a `catalogue.json` from *published* content,
   **atomically** writes it to storage, and records the run.
4. The **viewer** reads only that published `catalogue.json` (plus search/filter endpoints) and renders
   a Netflix-style browse experience.

## Goals / non-goals

**Goals**
- Correct, deterministic catalogue publication with **atomic** writes.
- Enforced validation: artwork dimensions/aspect/weight, `(content_group, language)` uniqueness,
  "published ⇒ has artwork + duration", "published show ⇒ has section".
- Enforced roles (editor vs admin).
- Operable pipeline: `docker-compose up` works first try, meaningful CI, secrets documented.

**Non-goals (v1)**
- Real video streaming / DRM.
- Multi-tenant CMS.
- Arbitrary language support (only `en`, `hi` in seed; schema is extensible).
- Aesthetics — usability over beauty.

## Repository layout (as intended)

```
peblo/
├── backend/          # FastAPI app, models, storage, publish job, tests
│   ├── app/
│   │   ├── api/          # routers (admin, catalog, health)
│   │   ├── core/         # config, security, roles
│   │   ├── db/           # engine, session, models
│   │   ├── storage/      # StorageBackend + LocalDisk / CloudflareR2
│   │   ├── publish/      # catalogue builder + atomic writer
│   │   └── schemas/      # Pydantic models
│   ├── alembic/          # migrations
│   └── tests/
├── cms/              # internal CMS (React + TS + TanStack Query)
├── viewer/           # browse UI (React + TS), reads only /catalog*
├── scripts/          # seed loader
├── docker-compose.yml
├── .github/workflows/ci.yml
├── .env.example
└── docs/             # ← this documentation
```

## Key constraints from `reference.json`

- **Sections**: `featured`, `series`, `minisodes`, `songs`.
- **Categories** (15): `adventure`, `folk`, `friendship`, `india`, `language`, `learning`, `maths`,
  `music`, `nature`, `reading`, `science`, `singalong`, `stories`, `travel`, `values`.
- **Languages**: `en`, `hi`.
- **Artwork specs** (all max 200 KB):

| Kind | Aspect | Target px |
|---|---|---|
| poster | 2:3 | 600×900 |
| banner | 16:9 | 1280×720 |
| thumbnail | 16:9 | 640×360 |
