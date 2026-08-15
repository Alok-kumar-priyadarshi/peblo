# Seed Data Analysis

Source: `seed_shows.json` — **95 episode rows across 8 shows**. The challenge states the seed data is
**deliberately imperfect**; the validation report should surface whatever we find. This file records
what the analysis found so implementers know what to handle and test.

## Composition

| Dimension | Values found |
|---|---|
| Sections | `featured`, `series`, `minisodes`, `songs` (all 4 allowed values present) |
| Categories | all 15 allowed values |
| Languages | `en`, `hi` |
| Statuses | `draft`, `published` |
| Artwork kinds | `poster`, `banner`, `thumbnail` |

## Shows

| Show | Slug | Section | Seasons | Episodes |
|---|---|---|---|---|
| Moti's Many Lives | `motis-many-lives` | featured | 0, 1 | 18 |
| Curious Cubs | `curious-cubs` | series | 1 | 8 |
| Discover India with Moti | `discover-india-with-moti` | minisodes | 1 | 10 |
| Number Nest | `number-nest` | series | 1 | 8 |
| Peblo Songs | `peblo-songs` | songs | 1 | 16 |
| Peblo Songs — Lyrical | `peblo-songs-lyrical` | songs | 1 | 10 |
| Rhyme Rangers | `rhyme-rangers` | **null** | 1 | 8 |
| Tiny Tales by Banyan Dadi | `tiny-tales-banyan-dadi` | series | 0, 1 | 17 |

## Imperfections found (the "gotchas")

| # | Issue | Affected rows | Consequence |
|---|---|---|---|
| 1 | **Published episode with no artwork** (`artwork_available = []`) | `ep_0036` (Discover India with Moti, s1e4) | Blocks publish — violates "published ⇒ has artwork" |
| 2 | **Show with `section = null`** | `Rhyme Rangers` (all 8 episodes, `ep_0085`–`ep_0092`) | Blocks publish — violates "published show ⇒ has section" |
| 3 | **Duplicate `(content_group, language)`** | `('motis-many-lives-s01e02', 'hi')` → `ep_0004` **and** `ep_9001` | Violates uniqueness invariant; must be reconciled during seed import |
| 4 | **Season 0 (trailers) present** | `ep_0093`, `ep_0094` | Must be excluded from normal season rendering |
| 5 | Same episode in `en`/`hi` with **differing durations** | e.g. `ep_0001` (510s) vs `ep_0002` (480s) | Expected (language variants); duration is per-variant |

## Implications for the seed/import process

The **seed loader** must:
1. **Reject or quarantine** rows that violate invariants (duplicate content_group+language) with a
   clear, human-readable message — do not silently drop or double-insert.
2. **Import `null` sections as `null`** so the validation report flags `Rhyme Rangers` honestly rather
   than defaulting it.
3. **Map `artwork_available` in seed** onto real `artworks` rows only where a corresponding file is
   provided in `assets/`; missing files become validation-report items, not errors.
4. Keep **Season 0** rows as normal episodes (season number 0) so the viewer's "hide trailers" rule is
   exercised by real data.

## Test fixtures to derive from this

- `ep_0036` → assert publish is blocked with "missing artwork" reason.
- `Rhyme Rangers` → assert publish is blocked with "missing section" reason.
- duplicate `motis-many-lives-s01e02/hi` → assert import surfaces a uniqueness violation.
- `motis-many-lives-s01e01` (en+hi) → assert catalogue collapses to one entry with `languages: ["en","hi"]`.
