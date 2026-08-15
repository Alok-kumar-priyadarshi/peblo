# Frontend Standards (TypeScript / React)

## Tooling

- **TypeScript** strict mode (`strict: true`), no `any` without justification.
- **Lint:** ESLint + `@typescript-eslint`; format with Prettier.
- **Build:** Vite.
- **Server state:** TanStack Query (the challenge's default; state an alternative + why if different).

## Structure (both apps)

```
src/
├── api/          # typed client (generated from OpenAPI where possible)
├── components/   # presentational + feature components
├── pages/        # routed screens
├── hooks/        # custom hooks / TanStack Query hooks
├── lib/          # utilities
└── types/        # shared types
```

## Conventions

- **camelCase** for vars/functions; **PascalCase** for components/types.
- One component per file; co-locate styles (CSS modules) with the component.
- **No business logic duplication** between CMS and Viewer; the API is the source of truth.
- Centralise API error handling: map error `code` → user-facing message in one place
  (see [`api/errors.md`](../api/errors.md)).

## State & data

- TanStack Query for server state; local `useState` only for transient UI.
- Never call admin endpoints from the Viewer; never fetch the whole catalogue to filter in the
  browser without a documented scale justification.
- Loading/empty/error/permission states are first-class (see
  [`architecture/frontend-architecture.md`](../architecture/frontend-architecture.md)).

## Accessibility & UX

- Form fields have labels and `aria` where needed; buttons have accessible names.
- Errors are human-readable and actionable for a non-engineer editor.
