# Backend API Documentation: RPG Player Character

**Version:** 1.0.0
**Base URL:** `https://api.dungeontoolbox.com/api/v1`
**Protocol:** HTTPS
**Content-Type:** `application/json`

---

## 1. Authentication & Security

All protected routes require a valid **Supabase JWT** in the Authorization header.

**Header Format:**
```http
Authorization: Bearer <SUPABASE_ACCESS_TOKEN>
```

**Common HTTP Status Codes:**
| Code | Meaning | Description |
| :--- | :--- | :--- |
| `200` | OK | Request succeeded. |
| `201` | Created | Resource successfully created. |
| `202` | Accepted | Async job started (e.g., PDF generation). |
| `400` | Bad Request | Validation failed (Zod error). |
| `401` | Unauthorized | Missing or invalid JWT token. |
| `402` | Payment Required | User has hit a strict limit (e.g., AI Credits). |
| `403` | Forbidden | User does not own the resource or hit a tier limit. |
| `429` | Too Many Requests | Rate limiting applied. |
| `500` | Server Error | Something went wrong on the backend. |

---

## 2. Character Management

### List User Characters (Search)

Returns a summary list of all characters owned by the authenticated user. Supports filtering.

* **Endpoint:** `GET /characters`
* **Query Params:**
    * `species` (string)
    * `className` (string)
    * `minLevel` (number)
    * `archetype` (string)
    * `tags` (string or array of strings)
* **Auth Required:** Yes

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "name": "Grom The Mighty",
      "totalLevel": 3,
      "species": "Half-Orc",
      "archetype": "Barbarian King",
      "description": "A fierce warrior seeking his throne.",
      "tags": ["melee", "king", "angry"],
      "xp": 3500,
      "classes": [
          { "className": "Barbarian", "classLevel": 3, "subclass": "Totem Warrior" }
      ],
      "is_public": true,
      "created_at": "2023-10-27T10:00:00Z"
    }
  ]
}
```

### Get Character Details

Retrieves the full JSON sheet for a specific character.

* **Endpoint:** `GET /characters/:id`
* **Auth Required:** Yes

**Response (200 OK):**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "name": "Grom The Mighty",
  "totalLevel": 3,
  "species": "Half-Orc",
  "archetype": "Barbarian King",
  "description": "A fierce warrior seeking his throne.",
  "tags": ["melee", "king", "angry"],
  "xp": 3500,
  "sheet_data": {
    "core": { "background": "Soldier" },
    "stats": { "str": 16, "dex": 12, "con": 14, "int": 8, "wis": 10, "cha": 12 },
    "vitals": { "hp_current": 25, "hp_max": 25 },
    "inventory": []
  },
  "classes": [
      { "className": "Barbarian", "classLevel": 3, "isPrimary": true }
  ],
  "owner_id": "user_uuid",
  "is_public": true
}
```

### Create Character

Creates a new character. Supports Multi-classing.
**Constraint:** Checks if Free Tier user has < 6 characters. If limit reached, returns `403`.

* **Endpoint:** `POST /characters`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "name": "New Hero",
  "total_level": 5,
  "species": "Dwarf",
  "background": "Blacksmith",
  "xp": 6500,
  "archetype": "Iron Defender",
  "description": "A stalwart defender of the mines.",
  "tags": ["tank", "dwarf", "metal"],
  "classes": [
      { "className": "Fighter", "classLevel": 3, "isPrimary": true },
      { "className": "Cleric", "classLevel": 2 }
  ],
  "sheet_data": {
    "vitals": { "hp_current": 45 }
  }
}
```

**Error (403 Forbidden):**

```json
{
  "error": "Limit Reached",
  "message": "Free tier users are limited to 6 characters. Please upgrade."
}
```

### Update Character (Partial)

Updates specific fields in the character sheet. Logic layer validates rules (e.g., cannot set HP > Max HP without override).

* **Endpoint:** `PATCH /characters/:id`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "total_level": 6,
  "xp": 14000,
  "tags": ["tank", "dwarf", "metal", "hero"],
  "sheet_data": {
    "vitals": { "hp_current": 15 }
  }
}
```

### Delete Character

Permanently deletes a character.

* **Endpoint:** `DELETE /characters/:id`
* **Auth Required:** Yes

---

## 3. Gameplay & Realtime (The "Logic Layer")

### Roll Dice

**Description:** Performs a server-side RNG roll using the **Universal Dice Syntax**. Supports standard notation (`1d20+5`) and advanced mechanics (`kh`, `dl`, `!`, `r`). If `characterId` is provided without a `formula`, it calculates the formula based on the character's stats.

* **Endpoint:** `POST /dice/roll`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "characterId": "uuid-string", // Optional. Used for context or stat lookup.
  "stat": "athletics",          // Optional. If provided, calculates formula from Character sheet.
  "formula": "2d6kh1 + 5",      // Optional. Overrides stat. Syntax: NdS[modifiers] (e.g. 4d6kh3, 1d20+5, 2d6!)
  "roll_mode": "normal"         // Optional: "normal" | "advantage" | "disadvantage"
}
```

**Response (200 OK):**

```json
{
  "formula": "2d6kh1 + 5",
  "result": 11,
  "breakdown": [
    {
      "type": "dice",
      "notation": "2d6kh1",
      "rolls": [2, 6],
      "kept": [6],
      "dropped": [2],
      "subtotal": 6
    },
    {
      "type": "static",
      "value": 5,
      "subtotal": 5
    }
  ],
  "rolls_meta": [] // Populated if advantage/disadvantage was used (contains [RollA, RollB])
}
```

*Note: The frontend should listen to `supabase.channel('room_characterID')` to display the visual roll.*

### Rest (Short/Long)

Applies healing logic, resets spell slots, and updates Hit Dice.

* **Endpoint:** `POST /game/rest`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "characterId": "uuid-string",
  "type": "short", // "short" | "long"
  "hit_dice_spent": 2 // Only for short rest
}
```

---

## 4. The Companion (AI & RAG)

### Chat / Rules Query

Sends a message to the AI. The backend injects character context and performs Vector Search on campaign notes before answering.

* **Endpoint:** `POST /companion/chat`
* **Auth Required:** Yes
* **Cost:** Deducts 1 AI Credit.

**Request Body:**

```json
{
  "characterId": "uuid-string", // Optional: Provides context about current hero
  "message": "Can I grapple the ghost?"
}
```

**Response (200 OK):**

```json
{
  "reply": "No, you cannot grapple a Ghost because it has the Incorporeal Movement trait...",
  "sources": ["SRD: Conditions", "My Adventure Notes (Session 3)"],
  "remaining_credits": 42
}
```

**Error (402 Payment Required):**

```json
{
  "error": "Quota Exceeded",
  "message": "You have used all 50 free AI interactions. Upgrade to Player Tier for more."
}
```

### Generate Character (Wizard)

Generates a full character sheet based on a prompt.

* **Endpoint:** `POST /companion/generate`
* **Auth Required:** Yes
* **Cost:** Deducts 1 AI Credit.

**Request Body:**

```json
{
  "prompt": "A spooky swamp druid who loves insects",
  "tone": "dark_fantasy",
  "level": 1
}
```

**Response (200 OK):**

```json
{
  "character": {
    "name": "Thistle Rotroot",
    "class": "Druid",
    "subclass": "Circle of Spores",
    "sheet_data": { ... }
  },
  "portrait_prompt": "A gnarly, moss-covered humanoid..." // Use this to query image gen if needed
}
```

---

## 5. Tools & Utilities (Microservices)

### Import from D&D Beyond

Parses a DDB Character JSON or PDF export and converts it to our schema.

* **Endpoint:** `POST /import/ddb`
* **Auth Required:** Yes
* **Content-Type:** `multipart/form-data` (if PDF) or `application/json`

**Request Body (JSON Mode):**

```json
{
  "ddb_json": { ...the_raw_ddb_data... }
}
```

### Export to PDF

Generates a printable PDF of the character sheet. This is an async job.

* **Endpoint:** `POST /export/pdf`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "characterId": "uuid-string",
  "theme": "classic" // "classic" | "modern"
}
```

**Response (202 Accepted):**

```json
{
  "job_id": "job_12345",
  "status": "processing",
  "status_url": "/api/v1/export/status/job_12345"
}
```

### Check Export Status

Polls the status of the PDF generation job.

* **Endpoint:** `GET /export/status/:jobId`
* **Auth Required:** Yes

**Response (200 OK):**

```json
{
  "status": "completed",
  "download_url": "[https://cdn.dungeontoolbox.com/pdfs/grom_sheet.pdf](https://cdn.dungeontoolbox.com/pdfs/grom_sheet.pdf)"
}
```

### Community Library Search

Public route to search shared characters.

* **Endpoint:** `GET /search/charcters`
* **Auth Required:** No

**Query Parameters:**

* `class` (string)
* `level` (number)
* `query` (string) - Name search
* `page` (number) - Pagination

**Response:**

```json
{
  "results": [
    { "id": "...", "name": "Aragorn", "class": "Ranger", "level": 5, "author": "User123" }
  ],
  "page": 1,
  "total": 50
}
```

---

## 6. Realtime Events (WebSocket Reference)

The backend broadcasts these events to the Supabase Channel `room_{characterId}`.

### Event: `dice_roll`

```json
{
  "type": "broadcast",
  "event": "dice_roll",
  "payload": {
    "user_id": "uuid",
    "character_name": "Grom",
    "result": 19,
    "breakdown": "1d20 (14) + 5"
  }
}
```

### Event: `vitals_update`

Sent when HP, Slots, or other dynamic stats change via the API.

```json
{
  "type": "broadcast",
  "event": "vitals_update",
  "payload": {
    "hp_current": 12,
    "hp_max": 25,
    "temp_hp": 0
  }
}
```