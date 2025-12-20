```markdown
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

### List User Characters

Returns a summary list of all characters owned by the authenticated user.

* **Endpoint:** `GET /characters`
* **Auth Required:** Yes

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "name": "Grom The Mighty",
      "class": "Barbarian",
      "level": 3,
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
  "sheet_data": {
    "core": { "race": "Human", "background": "Soldier" },
    "stats": { "str": 16, "dex": 12, "con": 14, "int": 8, "wis": 10, "cha": 12 },
    "vitals": { "hp_current": 25, "hp_max": 25 },
    "inventory": []
  },
  "owner_id": "user_uuid",
  "is_public": true
}

```

### Create Character

Creates a new character.
**Constraint:** Checks if Free Tier user has < 6 characters. If limit reached, returns `403`.

* **Endpoint:** `POST /characters`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "name": "New Hero",
  "class": "Fighter",
  "level": 1,
  "sheet_data": {
    "core": { "race": "Dwarf", "background": "Blacksmith" }
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

Performs a server-side RNG roll, applies modifiers, saves the log, and **broadcasts** the result via Supabase Realtime.

* **Endpoint:** `POST /game/roll`
* **Auth Required:** Yes

**Request Body:**

```json
{
  "characterId": "uuid-string",
  "type": "skill", // "skill" | "save" | "attack" | "damage" | "raw"
  "stat": "athletics", // skill name, ability name, or "1d8+2"
  "advantage_state": "normal" // "normal" | "advantage" | "disadvantage"
}

```

**Response (200 OK):**

```json
{
  "result": 19,
  "natural": 14,
  "modifier": 5,
  "is_crit": false,
  "broadcast_sent": true
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

* **Endpoint:** `GET /library/search`
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

```

```