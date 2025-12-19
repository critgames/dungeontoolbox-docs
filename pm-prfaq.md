Here is the complete PR/FAQ document in Markdown format, incorporating all the specific sections, pricing structures, and tonal adjustments we discussed.

---

# PR/FAQ: RPG Player Character by Crit Games

## Press Release

**FOR IMMEDIATE RELEASE**

**Crit Games Launches "RPG Player Character" on DungeonToolbox.com: The Faster, Easier, Better Way to Roleplay**

**SAMMAMISH, WA** — Crit Games is proud to announce the release of **RPG Player Character**, a revolutionary new tool available today at [dungeontoolbox.com](https://dungeontoolbox.com). Designed to eliminate the friction of complex mechanics, this tool empowers players to create characters in minutes rather than hours and enhances the gameplay experience for veterans and newcomers alike.

For too long, tabletop RPG players have bogged down in rulebooks, struggling to calculate stats before they can even roll a die. **RPG Player Character** solves this by putting the focus back on the story. It is the only character sheet that updates itself as you play.

**Create Quickly. Get to Playing.**
For the new adventurer, the barrier to entry is gone. With **The Companion**, users can go from a vague concept to a fully realized, stats-ready hero in moments. The tool handles the math and the rules, allowing players to focus on who their character is, not just what numbers are on the page.

**Enhance Your Game.**
For the inveterate roleplayer, **RPG Player Character** is the ultimate campaign companion. It organizes your roster of heroes and streamlines gameplay, making it easier to track abilities, inventory, and progression. It isn’t just about making the character; it’s about playing them better.

"We wanted to build something that respects the player's time," says the Founder of Crit Games. "Whether you are rolling your first dwarf fighter or managing your tenth high-level wizard, our goal is to make the process faster, easier, and better. We want you at the table playing, not stuck in a menu."

**RPG Player Character** is available now with a robust Free Tier and a Premium Tier for power users.

**Availability:**
Available immediately at [dungeontoolbox.com](https://dungeontoolbox.com).

### About Crit Games

Crit Games is dedicated to modernizing the tabletop experience, building digital tools that enhance the magic of pen-and-paper roleplaying.

---

## Product Requirements

| ID | Feature | Requirement | User Tier |
|:---|:---|:---|:---|
| FR-01 | Character Capacity | System must allow creation and storage of up to **6 characters**. | Free ($0/mo) |
| FR-02 | Companion Creation | System must provide **The Companion** to generate full character stats from natural language concepts. | Free ($0/mo) |
| FR-03 | Pre-Generated Library | System must provide access to a library of ready-to-play characters. | Free ($0/mo) |
| FR-04 | Limited Companion Usage | **The Companion** is limited to **3 Character Generations per month** for free users. | Free ($0/mo) |
| FR-05 | Extended Capacity | System must allow creation and storage of up to **12 characters**. | Player ($5/mo) |
| FR-06 | Companion Access | System must provide persistent **Companion** access for ongoing rules queries and inventory management. | Player ($5/mo) |
| FR-07 | Companion Memory | **The Companion** must index and retrieve context from user-provided Adventure Notes/Journals. | Player ($5/mo) |
| FR-08 | Print to PDF | System must generate a printable, formatted character sheet PDF. | Player ($5/mo) |
| FR-09 | Priority Access | Users must receive early access to new features. | Player ($5/mo) |
| FR-10 | Unlimited Management | System must allow unlimited character generation. | GM (Coming Soon) |
| FR-11 | D&D Beyond Import | System must import character data (JSON/PDF) from D&D Beyond. | Free ($0/mo) |

---

## Customer User Journeys

### Journey 1: The New Player (Free User)

**Persona:** Alex, a first-time D&D player.

1. **Discovery:** Alex finds **RPG Player Character** on dungeontoolbox.com while looking for help building a Druid.
2. **Onboarding:** Alex signs up for the Free Tier.
3. **Creation:** Alex selects "Create New Character." **The Companion** asks, "What kind of Druid do you imagine?" Alex types, "A spooky swamp druid who loves insects."
4. **The Assist:** **The Companion** suggests appropriate stats, a "Circle of Spores" subclass, and a background. Alex clicks "Approve."
5. **Completion:** Within 5 minutes, Alex has a level 1 Druid.
6. **Limit:** Two weeks later, Alex creates 3 more characters. When attempting to create a 4th character *with The Companion* in the same month, a prompt appears: *"You have reached your monthly limit of 3 Companion generations. Wait until next month or upgrade to the Player Tier for unlimited generations!"* (Note: He can still manually create characters up to his storage limit of 6).

### Journey 2: The Veteran (The Switcher)

**Persona:** Sarah, an experienced player with years of data in D&D Beyond.

1. **The Switch:** Sarah is tired of tabbing between her character sheet and the rules. She sees an ad: *"Import your DDB character in 1 click."*
2. **Import:** She exports her Level 5 Paladin from DDB as a PDF and drops it into **RPG Player Character**. The system parses the data instantly.
3. **Active Play (The "Changer"):** During combat, she types natural language into **The Companion's** command line: *"I cast Blinding Smite on the Orc."*
4. **Dynamic Update:** **The Companion** automatically deducts the spell slot and updates the chat log.
5. **Context Query:** Later, she forgets a plot point. She asks: *"Do I have any items that would help me bribe the guard mentioned in my Last Session notes?"* **The Companion** checks her inventory AND her notes, replying: *"Yes, you have the **Golden Signet Ring** you looted from the Baron's vault in session 3."*
6. **Validation:** Sarah realizes she didn't have to search through messy notebooks. She can just play.


## Detailed Feature Breakdown

### Creation & Management
*   **The Companion (Creation):** Go from a text description ("Spooky swamp druid") to a full stat block in under 2 minutes.
*   **D&D Beyond Import:** One-click import capability for users migrating from other platforms (supports PDF & JSON).
*   **Intelligent Sheet:** A mobile-responsive sheet that hides complexity until you need it, organizing actions by "Combat", "Exploration", and "Social".

### Active Play Assistance
*   **The Companion (Rules):** Ask natural language questions ("What's the range of Fireball?") and get instant, accurate answers cited from the System Reference Document (SRD).
*   **Dynamic State Updates:** Type "I take 15 slashing damage" and **The Companion** automatically adjusts HP and concentration checks.
*   **The Companion (Memory):** The AI indexes your adventure notes (Journal/Session Logs) to answer context-heavy questions ("Who is the Mayor?").

### Tools & Utilities
*   **Context-Aware Dice Roller:** Click any stat to roll. The AI adds modifiers automatically.
*   **Printable PDF Export:** Generate a classic-style character sheet for physical table play.

---


## FAQ

### External FAQ (Customer Facing)

**Q: Is the Free Tier really free?**
**A:** Yes. You can create, store, and play with up to 6 characters forever without paying a cent. You also get access to our library of Pre-Generated characters.

**Q: How does The Companion help me?**
**A:** For free users, **The Companion** acts as a creator for character generation (limited to 3/month). For paid users, **The Companion** becomes a persistent **Campaign Memory** agent. It understands the rules, your character stats, AND your personal notes, allowing it to answer complex context-aware questions.

**Q: Can I print my character sheet?**
**A:** The "Print to Sheet" feature, which formats your data into a standard tabletop layout, is available in the $5 Player Tier. Free users can view their character stats digitally on any device.

**Q: What happens if I stop paying the $5 subscription?**
**A:** Your account will revert to the Free Tier. You will keep your characters, but due to the AI generation limit, you will only be able to use **The Companion** to generate 3 new characters per month. You can still manually edit/play up to 6 active characters.

### Internal FAQ (Stakeholder Facing)

**Q: How do we define the AI limit for Free Users technically?**
**A:** We use a simple counter: `generations_this_month`. Free users get 3. This resets on the 1st of the month. This is technically simpler than managing "active slots" and encourages users to pay for *convenience* (speed) rather than just *storage*.

**Q: Why 6 characters for the free tier?**
**A:** Giving 6 slots (double the industry standard of 3) creates a massive "Generosity Hook" for marketing. We win on *storage*, but we monetize on *speed* (AI generation) and *intelligence* (Context Memory). This differentiates us from D&D Beyond.

**Q: What is the cost impact of The Companion?**
**A:** **The Companion** uses a structured chain of prompts. By caching pre-generated options and limiting the free user to creation-only prompts (rather than open-ended chat), we control inference costs while still providing the "magic" experience.

---


## Appendix: The Companion's Creation Flow (D&D 2024)

**The Companion** guides the user through these decisions. At every step, the user can select a pre-defined option or type a custom request.

### Step 0: Set the Tone
*Question:* "What is the tone of your campaign?"
*   [High Fantasy / Heroic Fantasy] (Standard D&D, Heroes vs Evil)
*   [Dark Fantasy / Gothic Horror] (Gritty, Ravenloft/Witcher style)
*   [Epic / Mythic Fantasy] (Save the World, Theros/Gods)
*   [Political Intrigue & Mystery] (City-based, Eberron/Candlekeep)
*   [Low Fantasy / Sword & Sorcery] (Rare Magic, Gritty Survival)
*   [Specialized (Swashbuckling / War)] (Pirates, Military)
*   *[Custom Input]*

### Step 1: Choose Your Class
*Question:* "What adventurer class defines your style?"
*   [Barbarian]
*   [Bard]
*   [Cleric]
*   [Druid]
*   [Fighter]
*   [Monk]
*   [Paladin]
*   [Ranger]
*   [Rogue]
*   [Sorcerer]
*   [Warlock]
*   [Wizard]
*   *[Custom Input]*

### Step 2: Choose Your Species
*Question:* "What is your heritage?"
*   [Aasimar]
*   [Dragonborn]
*   [Dwarf]
*   [Elf]
*   [Gnome]
*   [Goliath]
*   [Halfling]
*   [Human]
*   [Orc]
*   [Tiefling]
*   *[Custom Input]*

### Step 3: Choose Your Background
*Question:* "Where did you come from? (Grants Ability Scores & Feat)"
*   [Acolyte] (Int/Wis/Cha)
*   [Artisan] (Str/Dex/Int)
*   [Charlatan] (Dex/Con/Cha)
*   [Criminal] (Dex/Con/Int)
*   [Entertainer] (Str/Dex/Cha)
*   [Farmer] (Str/Con/Wis)
*   [Guard] (Str/Int/Wis)
*   [Guide] (Dex/Con/Wis)
*   [Hermit] (Con/Wis/Cha)
*   [Merchant] (Con/Int/Cha)
*   [Noble] (Str/Int/Cha)
*   [Sage] (Con/Int/Wis)
*   [Sailor] (Str/Dex/Wis)
*   [Scribe] (Dex/Int/Wis)
*   [Soldier] (Str/Dex/Con)
*   [Wayfarer] (Dex/Wis/Cha)
*   *[Custom Input]*

### Step 4: Choose Languages (If Applicable)
*Question:* "What languages do you speak?"
*   [Common] (Default)
*   [Elvish]
*   [Dwarvish]
*   [Draconic]
*   [Celestial]
*   [Abyssal]
*   [Thieves' Cant] (Rogue)
*   [Druidic] (Druid)
*   *[Custom Input]*

### Step 5: Level
*Question:* "What level is this character starting at?"
*   [Level 1]
*   [Level 3]
*   [Level 5]
*   *[Custom Input: 1-20]*

### Step 6: Ability Scores
*Question:* "How should we generate your stats?"
*   [Standard Array] (15, 14, 13, 12, 10, 8)
*   [Point Buy] (27 Points)
*   [Manual Input]

### Step 7: Equipment
*Question:* "What gear do you start with?"
*   [Starting Gold]
*   [Class Config A] (Default Weapons/Pack)
*   [Class Config B] (Alternative options)
*   *[Custom Input]*

### Step 8: Choose Spells (If Applicable)
*Question:* "What magic do you wield?"
*   [Recommended List] (Role-based AI selection)
*   [Attack Focus] (Damage spells)
*   [Utility Focus] (Control/Buff spells)
*   [Support Focus] (Healing/Protection spells)
*   *[Custom Input]*

### Step 9: Choose Alignment
*Question:* "What is your moral compass?"
*   [Lawful Good]
*   [Neutral Good]
*   [Chaotic Good]
*   [Lawful Neutral]
*   [True Neutral]
*   [Chaotic Neutral]
*   [Lawful Evil]
*   [Neutral Evil]
*   [Chaotic Evil]

### Step 10: Appearance (Optional)
*Question:* "What do you look like?"
*   [Generate Portrait Description] (Based on Class/Species/Tone)
*   [Visualize specific feature] ("A scar", "Glowing eyes")
*   *[Custom Input]*

### Step 11: Backstory (Optional)
*Question:* "What is your history?"
*   [Generate from Background] (Uses Step 3 Choice)
*   [Generate Tragic Event]
*   [Generate Heroic Origin]
*   *[Custom Input]*

### Step 12: Personality (Optional)
*Question:* "Who are you?"
*(Generates 2024-compliant Traits, Ideals, Bonds, and Flaws linked to Background)*
*   [Generate Suggested Traits]
*   [Select Archetype] (e.g., "The Grumpy Mentor", "The Naive Hero")
*   *[Custom Input]*

---

## Appendix: Example Character Data Structure (D&D 2024 Model)

*Note: This represents the backend JSON model visualized as Markdown.*

```markdown
# Character Sheet: Thorn 'Bug-Eater'
**Class:** Druid (Circle of Spores)  |  **Level:** 3  |  **Species:** Wood Elf
**Background:** Guide  |  **Alignment:** Chaotic Neutral  |  **XP:** 900

## Core Statistics
| Stat | Score | Mod | Save Prof? |
|:---:|:---:|:---:|:---:|
| **STR** | 10 | +0 | [ ] |
| **DEX** | 14 | +2 | [ ] |
| **CON** | 14 | +2 | [ ] |
| **INT** | 12 | +1 | [x] (+3) |
| **WIS** | 16 | +3 | [x] (+5) |
| **CHA** | 8 | -1 | [ ] |

## Vitals
*   **HP:** 24 / 24  (Hit Dice: 3d8)
*   **AC:** 14 (Leather + Shield)
*   **Initiative:** +2
*   **Speed:** 35 ft. (30 Base + 5 Wood Elf)
*   **Passive Perception:** 15
*   **Proficiency Bonus:** +2

## Death Saves
*   **Successes:** [ ] [ ] [ ]
*   **Failures:**  [ ] [ ] [ ]

## Features & Feats
### Class Features
*   **Druidic:** You know Druidic.
*   **Primal Order (Warden):** Proficiency in Medium Armor.
*   **Wild Shape:** Transform into beasts (CR 1/4).
*   **Circle of Spores:** Halo of Spores (Reaction 1d4 dmg), Symbiotic Entity (Temp HP + Dmg).

### Origin Feat (Background: Guide)
*   **Magic Initiate (Druid):** Learn 2 Cantrips, 1 Level 1 Spell.

### Species Traits (Wood Elf)
*   **Darkvision:** 60 ft.
*   **Fey Ancestry:** Adv vs Charm, immune to sleep magic.
*   **Trance:** 4 hour long rest.

## Proficiencies
*   **Armor:** Light, Medium, Shields.
*   **Weapons:** Simple Weapons, Scimitar, Shortsword, Longbow.
*   **Tools:** Herbalism Kit, Cartographer's Tools.
*   **Saving Throws:** Intelligence, Wisdom.
*   **Skills:**
    *   [x] Survival (+5)
    *   [x] Perception (+5)
    *   [x] Nature (+3)
    *   [x] Stealth (+4)
*   **Languages:** Common, Elvish, Druidic, Sylvan.

## Actions & Attacks
| Name | Bonus | Damage | Type | Mastery/Properties |
|:---|:---:|:---|:---|:---|
| **Scimitar** | +4 | 1d6 + 2 | Slashing | Nick, Light, Finesse |
| **Longbow** | +4 | 1d8 + 2 | Piercing | Slow, Ammunition (150/600), Two-Handed |
| **Produce Flame** | +5 | 1d8 | Fire | Range 30ft |
| **Halo of Spores**| DC 13 | 1d4 | Necrotic | Reaction (Start of creature's turn) |

## Spells (Spellcasting Ability: Wisdom)
**Spell Save DC:** 13  |  **Spell Attack:** +5
**Prepared Spells:** 5

*   **Cantrips:** Chill Touch, Druidcraft, Shillelagh, Guidance (Origin), Starry Wisp (Origin).
*   **Level 1 (4 Slots):** Entangle, Cure Wounds, Thunderwave, Goodberry (Origin).
*   **Level 2 (2 Slots):** Spike Growth, Wither and Bloom.

## Inventory
**Currency:** 15 GP, 4 SP

*   **Gear:** Leather Armor, Shield (Wooden), Scimitar, Longbow, Explorer's Pack, Druidic Totem (Sprig of Mistletoe).
*   **Magic Items:** None.

## Bio & Roleplay
*   **Traits:** I talk to bugs more than people.
*   **Ideal:** Balance – nature gives and takes.
*   **Bond:** I must protect the spore grove from the approaching city.
*   **Flaw:** I assume everyone civilized is corrupt.
*   **Appearance:** Messy moss-green hair, clothes covered in lichen.
```