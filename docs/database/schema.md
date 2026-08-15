# Database Schema (DDL)

PostgreSQL 16. Mirrors [`er-diagram.md`](./er-diagram.md). This is the canonical definition; Alembic
migrations should be derived from this.

```sql
-- Extensions
CREATE EXTENSION IF NOT EXISTS "pgcrypto";      -- gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS "pg_trgm";        -- optional, for future ILIKE acceleration

-- Enums (constrained vocabularies from reference.json)
CREATE TYPE user_role       AS ENUM ('editor', 'admin');
CREATE TYPE content_status  AS ENUM ('draft', 'published');
CREATE TYPE artwork_kind    AS ENUM ('poster', 'banner', 'thumbnail');
CREATE TYPE publish_status  AS ENUM ('running', 'succeeded', 'failed');

-- ---------------------------------------------------------------------------
-- users
-- ---------------------------------------------------------------------------
CREATE TABLE users (
    id            UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
    email         TEXT         NOT NULL UNIQUE,
    password_hash TEXT         NOT NULL,
    role          user_role    NOT NULL DEFAULT 'editor',
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT now()
);

-- ---------------------------------------------------------------------------
-- shows
-- ---------------------------------------------------------------------------
CREATE TABLE shows (
    id         UUID           PRIMARY KEY DEFAULT gen_random_uuid(),
    title      TEXT           NOT NULL,
    slug       TEXT           NOT NULL UNIQUE,
    synopsis   TEXT,
    -- validated against reference.json sections; NULL = blocked from publish
    section    TEXT,
    status     content_status NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ    NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ    NOT NULL DEFAULT now(),
    CONSTRAINT chk_shows_section
        CHECK (section IS NULL OR section IN ('featured','series','minisodes','songs'))
);

-- ---------------------------------------------------------------------------
-- seasons
-- ---------------------------------------------------------------------------
CREATE TABLE seasons (
    id         UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    show_id    UUID        NOT NULL REFERENCES shows(id) ON DELETE CASCADE,
    number     INTEGER     NOT NULL,          -- 0 = trailers (reserved)
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_seasons_show_number UNIQUE (show_id, number)
);

-- ---------------------------------------------------------------------------
-- episodes
-- ---------------------------------------------------------------------------
CREATE TABLE episodes (
    id               UUID           PRIMARY KEY DEFAULT gen_random_uuid(),
    season_id        UUID           NOT NULL REFERENCES seasons(id) ON DELETE CASCADE,
    number           INTEGER        NOT NULL,
    title            TEXT           NOT NULL,
    synopsis         TEXT,
    duration_seconds INTEGER,                  -- NULL until ready; required to publish
    language         TEXT           NOT NULL,  -- 'en' | 'hi'
    content_group    TEXT           NOT NULL,  -- collapses language variants
    status           content_status NOT NULL DEFAULT 'draft',
    created_at       TIMESTAMPTZ    NOT NULL DEFAULT now(),
    updated_at       TIMESTAMPTZ    NOT NULL DEFAULT now(),
    CONSTRAINT chk_episodes_language CHECK (language IN ('en','hi')),
    CONSTRAINT uq_episodes_season_number_lang UNIQUE (season_id, number, language),
    CONSTRAINT uq_episodes_content_group_lang UNIQUE (content_group, language)
);

-- ---------------------------------------------------------------------------
-- artworks  (poster/banner on a show; thumbnail on an episode)
-- ---------------------------------------------------------------------------
CREATE TABLE artworks (
    id          UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
    show_id     UUID         REFERENCES shows(id)    ON DELETE CASCADE,
    episode_id  UUID         REFERENCES episodes(id) ON DELETE CASCADE,
    kind        artwork_kind NOT NULL,
    object_key  TEXT         NOT NULL,
    width       INTEGER      NOT NULL,
    height      INTEGER      NOT NULL,
    size_bytes  INTEGER      NOT NULL,
    mime_type   TEXT         NOT NULL,
    sha256      TEXT         NOT NULL,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT now(),
    -- exactly one of show_id / episode_id
    CONSTRAINT chk_artworks_owner
        CHECK ((show_id IS NOT NULL)::int + (episode_id IS NOT NULL)::int = 1),
    -- one thumbnail per episode
    CONSTRAINT uq_artworks_episode_kind UNIQUE (episode_id, kind)
);

-- ---------------------------------------------------------------------------
-- publish_runs
-- ---------------------------------------------------------------------------
CREATE TABLE publish_runs (
    id           UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
    version      INTEGER       NOT NULL UNIQUE,   -- monotonic catalogue version
    triggered_by UUID          NOT NULL REFERENCES users(id),
    status       publish_status NOT NULL DEFAULT 'running',
    counts       JSONB         NOT NULL DEFAULT '{}',
    catalog_key  TEXT,                            -- e.g. catalogue/catalogue-{version}.json
    error        TEXT,
    started_at   TIMESTAMPTZ   NOT NULL DEFAULT now(),
    finished_at  TIMESTAMPTZ
);

-- ---------------------------------------------------------------------------
-- updated_at trigger (mutable tables)
-- ---------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION touch_updated_at() RETURNS trigger AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_users_updated    BEFORE UPDATE ON users    FOR EACH ROW EXECUTE FUNCTION touch_updated_at();
CREATE TRIGGER trg_shows_updated    BEFORE UPDATE ON shows    FOR EACH ROW EXECUTE FUNCTION touch_updated_at();
CREATE TRIGGER trg_episodes_updated BEFORE UPDATE ON episodes FOR EACH ROW EXECUTE FUNCTION touch_updated_at();
```

## Notes

- `section` is a `TEXT` with a CHECK rather than a Postgres enum so adding a section in
  `reference.json` doesn't require a migration. (Enums are used where the vocabulary is fixed.)
- `duration_seconds` is nullable because an episode may be drafted before a duration is known — but
  the "published ⇒ duration + artwork" invariant is enforced in the service layer (and test-covered)
  because it is a *cross-table* rule that a CHECK alone can't express.
- The two `UNIQUE` constraints on `episodes` implement invariant #1 both by position
  (`season, number, language`) and by identity (`content_group, language`).
