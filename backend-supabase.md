# Supabase Database Setup

This document describes the database schema required for the DungeonToolbox API and provides SQL commands to initialize it.

## Tables

### `characters`
Stores player characters.

| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | UUID | Primary Key (Default: `gen_random_uuid()`) |
| `user_id` | UUID | References `auth.users` (Supabase Auth) |
| `name` | Text | Character Name |
| `total_level` | Int | Character Total Level |
| `species` | Text | Character Species (formerly race) |
| `background` | Text | Character Background |
| `alignment` | Text | Character Alignment |
| `archetype` | Text | User defined Class Identity |
| `description` | Text | Elevator Pitch |
| `tags` | Text[] | Keywords (Searchable) |
| `xp` | Int | Experience Points |
| `data` | JSONB | Full Character Sheet Data |
| `created_at` | Timestamptz | Creation Date |
| `updated_at` | Timestamptz | Last Update |
| `is_public` | Boolean | Whether the character is searchable in the library |

### `game_logs`
Audit trail for dice rolls and game events.

| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | UUID | Primary Key |
| `session_id` | UUID | Optional Session ID |
| `user_id` | UUID | References `auth.users` |
| `character_id` | UUID | References `characters.id` |
| `event_type` | Text | "roll", "rest", "spell", etc. |
| `payload` | JSONB | Details (e.g., roll result, modifiers) |
| `created_at` | Timestamptz | Event Timestamp |

### `credits`
Tracks AI usage quotas.

| Column | Type | Description |
| :--- | :--- | :--- |
| `user_id` | UUID | Primary Key (References `auth.users`) |
| `balance` | Int | Remaining Credits |
| `last_reset` | Timestamptz | When the monthly quota was last reset |

---

## SQL Initialization Commands

Run these commands in the Supabase SQL Editor to set up your database.

```sql
-- DungeonToolbox: Create characters, game_logs, and credits tables with RLS and trigger

-- 1. Ensure extensions (no-op if already installed)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS vector;  -- optional for RAG / embeddings

-- 2. Characters table
CREATE TABLE IF NOT EXISTS public.characters (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name text NOT NULL,
  class text NOT NULL,
  level integer NOT NULL DEFAULT 1,
  data jsonb NOT NULL DEFAULT '{}'::jsonb,
  is_public boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- 3. Game logs table
CREATE TABLE IF NOT EXISTS public.game_logs (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  session_id uuid NULL,
  user_id uuid NULL REFERENCES auth.users(id) ON DELETE SET NULL,
  character_id uuid NULL REFERENCES public.characters(id) ON DELETE SET NULL,
  event_type text NOT NULL,
  payload jsonb NOT NULL DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- 4. Credits table
CREATE TABLE IF NOT EXISTS public.credits (
  user_id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  balance integer NOT NULL DEFAULT 50,
  last_reset timestamptz NOT NULL DEFAULT now()
);

-- 5. Enable Row Level Security
ALTER TABLE public.characters ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.game_logs ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.credits ENABLE ROW LEVEL SECURITY;

-- 6. Policies for characters
DO $$
BEGIN
  -- SELECT: own characters OR public characters
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'characters' AND p.policyname = 'characters_select_owner_or_public'
  ) THEN
    CREATE POLICY characters_select_owner_or_public
      ON public.characters
      FOR SELECT
      TO authenticated
      USING (
        (user_id = (SELECT auth.uid())) OR (is_public = true)
      );
  END IF;

  -- INSERT: must be inserting as self
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'characters' AND p.policyname = 'characters_insert_owner'
  ) THEN
    CREATE POLICY characters_insert_owner
      ON public.characters
      FOR INSERT
      TO authenticated
      WITH CHECK (user_id = (SELECT auth.uid()));
  END IF;

  -- UPDATE: only owner
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'characters' AND p.policyname = 'characters_update_owner'
  ) THEN
    CREATE POLICY characters_update_owner
      ON public.characters
      FOR UPDATE
      TO authenticated
      USING (user_id = (SELECT auth.uid()))
      WITH CHECK (user_id = (SELECT auth.uid()));
  END IF;

  -- DELETE: only owner
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'characters' AND p.policyname = 'characters_delete_owner'
  ) THEN
    CREATE POLICY characters_delete_owner
      ON public.characters
      FOR DELETE
      TO authenticated
      USING (user_id = (SELECT auth.uid()));
  END IF;
END$$;

-- 7. Policies for game_logs
DO $$
BEGIN
  -- SELECT: own logs
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'game_logs' AND p.policyname = 'game_logs_select_owner'
  ) THEN
    CREATE POLICY game_logs_select_owner
      ON public.game_logs
      FOR SELECT
      TO authenticated
      USING (user_id = (SELECT auth.uid()));
  END IF;

  -- INSERT: allow creating logs as the current user (or null for system logs)
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'game_logs' AND p.policyname = 'game_logs_insert_owner'
  ) THEN
    CREATE POLICY game_logs_insert_owner
      ON public.game_logs
      FOR INSERT
      TO authenticated
      WITH CHECK (
        user_id IS NULL OR user_id = (SELECT auth.uid())
      );
  END IF;
END$$;

-- 8. Policies for credits
DO $$
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'credits' AND p.policyname = 'credits_select_owner'
  ) THEN
    CREATE POLICY credits_select_owner
      ON public.credits
      FOR SELECT
      TO authenticated
      USING (user_id = (SELECT auth.uid()));
  END IF;

  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'credits' AND p.policyname = 'credits_update_owner'
  ) THEN
    -- Allow updates only to the owner (note: sensitive operations like decrementing balance should use server-side service role)
    CREATE POLICY credits_update_owner
      ON public.credits
      FOR UPDATE
      TO authenticated
      USING (user_id = (SELECT auth.uid()))
      WITH CHECK (user_id = (SELECT auth.uid()));
  END IF;

  IF NOT EXISTS (
    SELECT 1 FROM pg_policies p
    WHERE p.schemaname = 'public' AND p.tablename = 'credits' AND p.policyname = 'credits_insert_owner'
  ) THEN
    CREATE POLICY credits_insert_owner
      ON public.credits
      FOR INSERT
      TO authenticated
      WITH CHECK (user_id = (SELECT auth.uid()));
  END IF;
END$$;

-- 9. Trigger: update updated_at on characters
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
RETURNS trigger AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

DROP TRIGGER IF EXISTS update_characters_updated_at ON public.characters;
CREATE TRIGGER update_characters_updated_at
BEFORE UPDATE ON public.characters
FOR EACH ROW
EXECUTE FUNCTION public.update_updated_at_column();

-- 10. Helpful indexes
CREATE INDEX IF NOT EXISTS idx_characters_user_id ON public.characters (user_id);
CREATE INDEX IF NOT EXISTS idx_game_logs_user_id ON public.game_logs (user_id);
CREATE INDEX IF NOT EXISTS idx_game_logs_character_id ON public.game_logs (character_id);

-- UPDATE for MUTLI-CLASSING
-- 1. Update the main characters table to hold "Searchable" Metadata
-- We rename 'level' to 'total_level' to avoid confusion with class levels
ALTER TABLE public.characters 
  RENAME COLUMN level TO total_level;

ALTER TABLE public.characters
  DROP COLUMN class, -- We are moving this to a new table
  ADD COLUMN race text, -- Promoted from JSON for fast search
  ADD COLUMN background text, -- Promoted from JSON
  ADD COLUMN alignment text; -- Promoted from JSON

-- 2. Create the Multi-Classing Table
-- This handles: "Wizard 3 (Evocation)" AND "Fighter 2 (Battle Master)"
CREATE TABLE IF NOT EXISTS public.character_classes (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  character_id uuid NOT NULL REFERENCES public.characters(id) ON DELETE CASCADE,
  class_name text NOT NULL, -- e.g. "Wizard"
  class_level integer NOT NULL DEFAULT 1, -- e.g. 3
  subclass text, -- e.g. "School of Evocation"
  is_primary boolean DEFAULT false, -- To mark their "main" class
  created_at timestamptz DEFAULT now()
);

-- 3. Add Indexes for High-Speed Search
-- These make filters like "Find all Wood Elves" or "Find all Wizards" instant.
CREATE INDEX idx_characters_race ON public.characters(race);
CREATE INDEX idx_characters_total_level ON public.characters(total_level);
CREATE INDEX idx_character_classes_name ON public.character_classes(class_name);
CREATE INDEX idx_character_classes_subclass ON public.character_classes(subclass);

-- 4. Enable RLS on the new table
ALTER TABLE public.character_classes ENABLE ROW LEVEL SECURITY;

-- Allow public read access to classes (so library search works)
CREATE POLICY classes_select_public
  ON public.character_classes FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM public.characters 
      WHERE id = character_classes.character_id 
      AND (is_public = true OR user_id = auth.uid())
    )
  );

  -- 1. Rename 'race' to 'species' (Standardizing for 2024 rules)
ALTER TABLE public.characters 
  RENAME COLUMN race TO species;

-- 2. Rename the index for consistency (Optional but good practice)
-- Only run this if you created the index in the previous step
ALTER INDEX IF EXISTS idx_characters_race 
  RENAME TO idx_characters_species;

-- 3. Add the 'xp' column
ALTER TABLE public.characters 
  ADD COLUMN xp integer NOT NULL DEFAULT 0;

-- 4. Add an index for XP 
-- (Useful if you ever want a leaderboard like "High Score" or "Most Experienced")
CREATE INDEX IF NOT EXISTS idx_characters_xp ON public.characters(xp);

-- 1. Add the metadata columns
ALTER TABLE public.characters
  ADD COLUMN archetype text, -- "The user-defined 'Class Identity'"
  ADD COLUMN description text, -- "The elevator pitch"
  ADD COLUMN tags text[] DEFAULT '{}'; -- "The keywords"

-- 2. Add an index specifically for the Array Search (GIN Index)
-- This makes searching for "contains tag 'Peaceful'" lightning fast.
CREATE INDEX idx_characters_tags ON public.characters USING GIN (tags);

-- 3. Add index for archetype string search
CREATE INDEX idx_characters_archetype ON public.characters(archetype);

-- Added Lifetime Usage Column
ALTER TABLE credits
ADD COLUMN "lifetime_usage" INTEGER DEFAULT 0;


-- Character Notes
BEGIN;

-- Table
CREATE TABLE public.character_notes (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  character_id uuid NOT NULL REFERENCES public.characters(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES auth.users(id),
  note text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- Indexes
CREATE INDEX character_notes_character_created_idx ON public.character_notes (character_id, created_at DESC);
CREATE INDEX character_notes_user_idx ON public.character_notes (user_id);

-- Enable RLS
ALTER TABLE public.character_notes ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY "notes_select_owner" ON public.character_notes
  FOR SELECT
  TO authenticated
  USING ( user_id = (SELECT auth.uid()) );

CREATE POLICY "notes_insert_owner" ON public.character_notes
  FOR INSERT
  TO authenticated
  WITH CHECK ( user_id = (SELECT auth.uid()) );

CREATE POLICY "notes_update_owner" ON public.character_notes
  FOR UPDATE
  TO authenticated
  USING ( user_id = (SELECT auth.uid()) )
  WITH CHECK ( user_id = (SELECT auth.uid()) );

CREATE POLICY "notes_delete_owner" ON public.character_notes
  FOR DELETE
  TO authenticated
  USING ( user_id = (SELECT auth.uid()) );

-- Trigger function to auto-update updated_at
CREATE OR REPLACE FUNCTION public.set_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$;

CREATE TRIGGER tr_character_notes_updated_at
BEFORE UPDATE ON public.character_notes
FOR EACH ROW EXECUTE FUNCTION public.set_updated_at();

COMMIT;

-- Charcter Notes Added Title Column
ALTER TABLE public.character_notes
ADD COLUMN title text;

-- Optional: index to assist queries that filter by character and order by newest notes.
-- (Keeps index small by not indexing the full note text; title is included but may be large.)
CREATE INDEX IF NOT EXISTS character_notes_character_created_title_idx
  ON public.character_notes (character_id, created_at DESC, title);

-- If you want the title to be required for every note, use this instead:
-- ALTER TABLE public.character_notes ALTER COLUMN title SET NOT NULL;

-- Character Notes Embeddings
CREATE TABLE public.character_note_embeddings (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  note_id uuid NOT NULL REFERENCES public.character_notes(id) ON DELETE CASCADE,
  model text NOT NULL,
  embedding vector(768) NOT NULL,
  content text NOT NULL,
  chunk_index integer DEFAULT 0,
  metadata jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX idx_note_embeddings_note_id ON public.character_note_embeddings(note_id);
-- Example vector index (pgvector / supabase vector). Adjust operator class and method depending on extension:
-- For pgvector + ivfflat:
-- CREATE INDEX idx_embeddings_vector_ivfflat ON public.character_note_embeddings USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);


--- SETUP SUBSCRIPTONS
ALTER TABLE public.credits RENAME TO user_subscriptions;
ALTER TABLE public.user_subscriptions RENAME COLUMN balance TO credits;

ALTER TABLE public.user_subscriptions
ADD COLUMN stripe_customer_id text UNIQUE,
ADD COLUMN stripe_subscription_id text,
ADD COLUMN subscription_status text DEFAULT 'active',
ADD COLUMN tier text DEFAULT 'free',
ADD COLUMN current_period_end timestamptz,
ADD COLUMN updated_at timestamptz DEFAULT timezone('utc', now()),
ADD COLUMN metadata jsonb DEFAULT '{}'::jsonb;

ALTER TABLE public.user_subscriptions
ALTER COLUMN credits SET DEFAULT 50;

-- Optional: add indexes for faster lookups
CREATE INDEX IF NOT EXISTS idx_user_subscription_stripe_subscription_id ON public.user_subscriptions (stripe_subscription_id);
CREATE INDEX IF NOT EXISTS idx_user_subscription_tier ON public.user_subscriptions (tier);

-- Add a CHECK constraint for subscription_status
ALTER TABLE public.user_subscriptions
ADD CONSTRAINT user_subscription_status_check CHECK (subscription_status IN ('active','past_due','canceled','unpaid','trialing'));

--  Trigger Updated At
CREATE TRIGGER set_updated_at_on_user_subscriptions BEFORE UPDATE ON public.user_subscriptions FOR EACH ROW EXECUTE FUNCTION public.set_updated_at();

-- Adding Notes Cateogories and Status
-- 1) Create ENUM type for category
CREATE TYPE public.character_note_category AS ENUM (
  'NPC',
  'Quest',
  'Location',
  'Item',
  'Lore',
  'Log',
  'Other'
);

-- 2) Create ENUM type for status
CREATE TYPE public.character_note_status AS ENUM (
  'pinned',
  'archived'
);

-- 3) Add category column (non-null, default 'Log')
ALTER TABLE public.character_notes
ADD COLUMN category public.character_note_category NOT NULL DEFAULT 'Log';

-- 4) Add status column (nullable)
ALTER TABLE public.character_notes
ADD COLUMN status public.character_note_status;

-- 5) Optional: add indexes to speed queries filtering by these columns
CREATE INDEX IF NOT EXISTS idx_character_notes_category ON public.character_notes (category);
CREATE INDEX IF NOT EXISTS idx_character_notes_status ON public.character_notes (status);

-- Make NOTES searchable
-- 1) Install/uninstall not needed: pg_trgm and unaccent are already available.
CREATE EXTENSION IF NOT EXISTS unaccent WITH SCHEMA public;

-- 2) Add tsvector column for full-text search
ALTER TABLE public.character_notes
ADD COLUMN note_search tsvector GENERATED ALWAYS AS (
  to_tsvector('simple', coalesce(unaccent(note), ''))
) STORED;

-- 3) Create GIN index on the tsvector column for FTS queries
CREATE INDEX IF NOT EXISTS idx_character_notes_note_search ON public.character_notes USING GIN (note_search);

-- 4) Optional: create a pg_trgm GIN index to accelerate ILIKE / %search% and similarity searches
CREATE INDEX IF NOT EXISTS idx_character_notes_note_trgm ON public.character_notes USING GIN (note gin_trgm_ops);

-- 5) Optional: create a functional index for lower(unaccent(note)) to speed case-insensitive searches
CREATE INDEX IF NOT EXISTS idx_character_notes_note_unaccent_lower ON public.character_notes USING GIN (lower(unaccent(note)) gin_trgm_ops);

-- SETUP Search Vectors:
ALTER TABLE public.character_notes ADD COLUMN IF NOT EXISTS search_vector tsvector;

-- Populate initially
UPDATE public.character_notes SET search_vector =
  to_tsvector('simple', coalesce(title,'') || ' ' || coalesce(note,''));

-- Create index
CREATE INDEX IF NOT EXISTS idx_character_notes_search_vector ON public.character_notes USING GIN(search_vector);

-- Trigger function to update
CREATE OR REPLACE FUNCTION public.character_notes_search_vector_update() RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN
  NEW.search_vector := to_tsvector('simple', coalesce(NEW.title,'') || ' ' || coalesce(NEW.note,''));
  RETURN NEW;
END;
$$;

CREATE TRIGGER trg_character_notes_search_vector
BEFORE INSERT OR UPDATE ON public.character_notes
FOR EACH ROW EXECUTE FUNCTION public.character_notes_search_vector_update();

-- Enable Better Text Searchign with Stemming and Stop-Words
-- Corrected: update search_vector using english + unaccent with weights
UPDATE public.character_notes SET search_vector =
  setweight(to_tsvector('english', unaccent(coalesce(title,''))), 'A') ||
  setweight(to_tsvector('english', unaccent(coalesce(note,''))), 'B');

-- Drop and recreate a standard GIN index on the tsvector
DROP INDEX IF EXISTS idx_character_notes_search_vector;
CREATE INDEX idx_character_notes_search_vector ON public.character_notes USING GIN(search_vector);

-- Create updated trigger function
CREATE OR REPLACE FUNCTION public.character_notes_search_vector_update() RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN
  NEW.search_vector :=
    setweight(to_tsvector('english', unaccent(coalesce(NEW.title,''))), 'A') ||
    setweight(to_tsvector('english', unaccent(coalesce(NEW.note,''))), 'B');
  RETURN NEW;
END;
$$;

-- Recreate trigger
DROP TRIGGER IF EXISTS trg_character_notes_search_vector ON public.character_notes;
CREATE TRIGGER trg_character_notes_search_vector
BEFORE INSERT OR UPDATE ON public.character_notes
FOR EACH ROW EXECUTE FUNCTION public.character_notes_search_vector_update();

-- Added RPC function so that searching text is easier
CREATE OR REPLACE FUNCTION public.rpc_search_character_notes_by_character(
  p_query text,
  p_character_id uuid,
  p_limit int DEFAULT 20,
  p_offset int DEFAULT 0
)
RETURNS TABLE (
  total_count bigint,
  results jsonb
)
LANGUAGE plpgsql STABLE AS
$$
DECLARE
  q tsquery;
  rec RECORD;
  rows jsonb := '[]'::jsonb;
  cnt bigint;
  caller uuid := (select auth.uid());
  owner uuid;
BEGIN
  IF p_query IS NULL OR btrim(p_query) = '' THEN
    total_count := 0;
    results := rows;
    RETURN;
  END IF;
  IF p_character_id IS NULL THEN
    RAISE EXCEPTION 'p_character_id is required';
  END IF;

  -- Verify character exists and is owned by caller
  SELECT owner_id INTO owner
  FROM public.characters
  WHERE id = p_character_id;

  IF owner IS NULL THEN
    RAISE EXCEPTION 'character_id not found';
  END IF;

  IF caller IS NULL OR caller <> owner THEN
    RAISE EXCEPTION 'permission denied';
  END IF;

  q := plainto_tsquery('english', unaccent(p_query));

  SELECT count(*) INTO cnt
  FROM public.character_notes cn
  WHERE cn.character_id = p_character_id
    AND cn.search_vector @@ q;

  total_count := cnt;

  FOR rec IN
    SELECT
      cn.id,
      ts_rank_cd(cn.search_vector, q) AS rank,
      cn.title,
      cn.note,
      ts_headline('english', cn.note, q,
        'StartSel=<mark>,StopSel=</mark>,MaxFragments=2,ShortWord=3,FragmentDelimiter=...') AS snippet,
      cn.created_at
    FROM public.character_notes cn
    WHERE cn.character_id = p_character_id
      AND cn.search_vector @@ q
    ORDER BY rank DESC, cn.created_at DESC
    LIMIT p_limit OFFSET p_offset
  LOOP
    rows := rows || jsonb_build_array(
      jsonb_build_object(
        'id', rec.id,
        'rank', rec.rank,
        'title', rec.title,
        'note', rec.note,
        'snippet', rec.snippet,
        'created_at', rec.created_at
      )
    );
  END LOOP;

  results := rows;
  RETURN;
END;
$$;

```
