# Technical Design Document: Node.js API for RPG Player Character

**Role:** Backend Engineer
**Task:** Scaffold a Node.js API Service
**Stack:** Node.js (Fastify), TypeScript, Supabase (DB/Auth), Gemini 1.5

---

## 1. System Architecture

The system uses a **Backend-for-Frontend (BFF)** pattern.
* **Frontend:** Next.js (manages UI).
* **Backend:** Node.js API (manages Game Rules & AI).
* **Database:** Supabase (PostgreSQL).
* **Auth:** Supabase Auth (Client sends JWT, Backend verifies it).
* **Security:** Use HTTPs, JWT, and Rate Limiting.

### Why this architecture?
We need strict validation of D&D 2024 rules before saving data. Doing this in a Node.js middleware layer is cleaner than using Database Triggers.

---

## 2. API Specifications (Node.js)

### 2.1 Technology Choices
* **Framework:** Fastify (for low overhead).
* **Language:** TypeScript.
* **ORM:** Prisma (connected to Supabase Postgres).
* **Validation:** Zod.

### 2.2 Middleware: Auth Guard
The API must verify the `Authorization: Bearer <token>` header on every protected route.
* Use `server.decorate` to attach the `user_id` to the request object.
* If the token is invalid or expired, return `401 Unauthorized`.

### 2.3 Key Endpoints

#### **POST** `/api/characters/generate` (The Companion)
* **Input:** `{ prompt: "Spooky Druid", tone: "Dark Fantasy" }`
* **Logic:**
    1.  Check User's "AI Credit" balance in DB.
    2.  If strictly > 0, call **Google Gemini 1.5 Flash**.
    3.  System Prompt: "You are a D&D 5.5e generator..."
    4.  Parse JSON response.
    5.  Deduct 1 Credit.
    6.  **Do NOT save to DB yet.** Return JSON to client for preview.

#### **POST** `/companion/chat`
* **Input:** `{ message: "Can I grapple a ghost?", characterId: "uuid" }`
* **Logic:**
    1.  **Auth Guard:** Verify user.
    2.  **Credit Check:** Deduct 1 AI Credit (if applicable).
    3.  **Context Retrieval:** Fetch character sheet + relevant vector search results (RAG).
    4.  **AI Call:** Send context + user message to Gemini 1.5.
    5.  **Response:** Return AI answer + source citations.

#### **GET** `/library/characters`
- Public endpoint for the "Community Library".
- Returns a flattened "Card View" (id, name, level, archetype, description, tags, class summary).
- **Optimization**: Explicitly excludes `sheet_data` JSONB column to reduce payload size.
- **Filtering**: Supports fuzzy text (`q`), exact tags (`hasEvery`), level range, and class filtering.

#### **POST** `/api/characters/save`
* **Input:** `{ characterData: JSON }`
* **Logic:**
    1.  **Rule Check:** Validate that `stats` are within legal limits.
    2.  **Schema Check:** Ensure `species`, `xp`, `total_level` are provided for indexing.
    3.  **Limit Check:** Count user's existing characters. If `free` tier and count >= 6, throw `403`.
    4.  **Persist:** Use Prisma to write to `characters` table.

#### **POST** `/api/dice/roll` (Hybrid Realtime)
* **Input:** `{ characterId, stat: "athletics" }`
* **Logic:**
    1.  Calculate roll (Server-side RNG).
    2.  Save result to `game_logs` table (Audit trail).
    2.  Save result to `game_logs` table (Audit trail).
    3.  **Broadcast:** Use `@supabase/supabase-js` admin client to push a "Broadcast" event to the specific `room_id` channel so all clients see the roll.

### 2.4 The Universal Dice Syntax (Logic Layer)

The API implements a robust recursive parser for dice formulas, supporting complex mechanics beyond simple arithmetic.

**Features:**
*   **Atomic Structure:** `[Count]d[Sides][Modifiers]`
*   **Keep/Drop:** `kh` (Keep Highest), `kl` (Keep Lowest), `dh` (Drop Highest), `dl` (Drop Lowest). Example: `4d6kh3`.
*   **Exploding:** `!` (Reroll if max).
*   **Reroll:** `r{target}` (Reroll if equal to target).
*   **Meta-State:** Handles Advantage (Run Twice, Keep High) and Disadvantage (Run Twice, Keep Low) at the "Instance" level.

**JSON Output:**
Returns a detailed log of every die rolled, kept, dropped, and summed, ensuring full transparency for the VTT.

---

## 3. Microservice Interface (The "Worker")

*If we determine PDF generation is too heavy for the main API, we offload it.*

**Service:** `pdf-worker`
**Communication:** Redis Queue (BullMQ)

**Flow:**
1.  Node.js API receives `POST /export/pdf`.
2.  Adds job to `pdf_queue`.
3.  Worker (running Puppeteer or WeasyPrint) picks up job.
4.  Generates PDF -> Uploads to Supabase Storage -> Updates Job Status.
5.  Client polls for download URL.

---

## 4. Implementation Steps for AI

1.  **Scaffold:** Initialize Fastify project with TypeScript and Zod.
2.  **Database:** Run `npx prisma init` and pull schema from Supabase.
3.  **Auth:** Create the JWT verification decorator.
4.  **Route - Chat:** Create the Gemini integration route.
5.  **Route - Roll:** Create the dice rolling logic and Supabase Realtime broadcast.