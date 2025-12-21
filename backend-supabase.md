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

```