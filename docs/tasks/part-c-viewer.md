# Part C — Viewer Tasks

## T-VW-01 — Home (hero + rows)
- **Acceptance:** Netflix-style home with a featured hero (banner) and horizontal rows per section
  (poster cards), reading only `GET /catalog`.
- **Reference:** [`sequences/browse-flow.md`](../sequences/browse-flow.md).

## T-VW-02 — Search + filters
- **Acceptance:** search + filters (category, language) with a sensible empty state, using
  `GET /catalog/search`.
- **Reference:** [`sequences/search-flow.md`](../sequences/search-flow.md).

## T-VW-03 — Show detail
- **Acceptance:** synopsis, seasons and episodes (thumbnails), language options for grouped episodes,
  and trailers (Season 0) excluded from normal seasons.
- **Reference:** [`diagrams/catalogue-shape.md`](../diagrams/catalogue-shape.md).

## T-VW-04 — Slow-image handling
- **Acceptance:** pleasant on slow networks — lazy loading + blur-up/placeholder colors, no layout
  shift.
- **Reference:** [`architecture/frontend-architecture.md`](../architecture/frontend-architecture.md).
