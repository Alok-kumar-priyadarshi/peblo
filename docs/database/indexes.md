# Indexes

## Declared

| Index | Table | Columns | Serves |
|---|---|---|---|
| `uq_shows_slug` (unique) | shows | `slug` | slug lookup; deterministic ordering |
| `uq_seasons_show_number` (unique) | seasons | `(show_id, number)` | season navigation + uniqueness |
| `uq_episodes_season_number_lang` (unique) | episodes | `(season_id, number, language)` | episode uniqueness by position |
| `uq_episodes_content_group_lang` (unique) | episodes | `(content_group, language)` | invariant #1; publish collapse lookup |
| `uq_artworks_episode_kind` (unique) | artworks | `(episode_id, kind)` | one thumbnail per episode |
| `uq_publish_runs_version` (unique) | publish_runs | `version` | version ordering / rollback |

## Recommended (add via migration if volume grows)

| Index | Columns | Why |
|---|---|---|
| `ix_episodes_status` | `(status)` | filter "published" during publish/build |
| `ix_episodes_language` | `(language)` | search/filter by language |
| `ix_episodes_content_group` | `(content_group)` | collapse variants quickly |
| `ix_shows_status` | `(status)` | publish eligibility scan |
| `ix_shows_section` | `(section)` | group-by-section scan |
| `ix_artworks_show_id` | `(show_id)` | fetch a show's poster/banner |
| `ix_artworks_episode_id` | `(episode_id)` | fetch an episode's thumbnail |

## Search scaling note

`q` uses `ILIKE '%term%'` in v1, which **cannot** use a B-tree index. When catalogue size grows,
switch to a `tsvector` column + GIN index (see [`sequences/search-flow.md`](../sequences/search-flow.md)).
The `pg_trgm` extension (already provisioned in `schema.md`) enables GIN trigram indexes for
case-insensitive substring search as an intermediate step.
