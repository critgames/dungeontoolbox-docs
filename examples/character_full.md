{
  "meta": {
    "character_id": "ez-master-2024",
    "last_updated": "2025-12-27T11:45:00Z",
    "exported_date": "2025-12-27T11:45:00Z",
    "game_system": "dnd-2024",
    "schema_version": "1.0.0",
    "compatible_apps": ["dungeontoolbox"],
    "api_version": "v1"
  },
  "name": "Ezren",
  "level": 20,
  "xp": 355000,
  "species": "Elf",
  "lineage": "High Elf",
  "alignment": "Lawful Neutral",
  "background": "Sage",
  "classes": [
    {
        "name": "Bard",
        "class_level": 9,
        "isPrimary": true,
        "subclass": "College of Valor"
    },
    {
        "name": "Fighter",
        "class_level": 3,
        "isPrimary": false,
        "subclass": "Battle Master"
    }
  ],
  "vitals": {
    "ac": { "score": 12, "shield_equipped": false, "bonus": 0 },
    "hp": { "current": 102, "max": 102, "temp": 0 },
    "hit_dice": [
      { "type": "d8", "total": 9, "spent": 0 },
      { "type": "d10", "total": 3, "spent": 0 }
    ],
    "death_saves": { "successes": 0, "failures": 0 },
    "initiative": 2,
    "speed": 30,
    "passive_perception": 13,
    "proficiency_bonus": 6,
    "heroic_inspiration": true,
    "bardic_inspiration": "1d6",
    "exhaustion_level": 0
  },
  "abilities": {
    "str": { "score": 8, "mod": -1, "save": false },
    "dex": { "score": 14, "mod": 2, "save": false },
    "con": { "score": 14, "mod": 2, "save": false },
    "int": { "score": 20, "mod": 5, "save": true },
    "wis": { "score": 14, "mod": 2, "save": true },
    "cha": { "score": 10, "mod": 0, "save": false }
  },
    "skills": {
      "acrobatics": {"mod": 2, "ability": "DEX", "proficient": false, "expertise": false},
      "animal_handling": {"mod": 2, "ability": "WIS", "proficient": false, "expertise": false},
      "arcana": {"mod": 2, "ability": "INT", "proficient": false, "expertise": false},
      "athletics": {"mod": 10, "ability": "STR", "proficient": true, "expertise": false},
      "deception": {"mod": 9, "ability": "CHA", "proficient": true, "expertise": false},
      "history": {"mod": 2, "ability": "INT", "proficient": false, "expertise": false},
      "insight": {"mod": 4, "ability": "WIS", "proficient": false, "expertise": false},
      "intimidation": {"mod": 13, "ability": "CHA", "proficient": true, "expertise": false},
      "investigation": {"mod": 2, "ability": "INT", "proficient": false, "expertise": false},
      "medicine": {"mod": 4, "ability": "WIS", "proficient": false, "expertise": false},
      "nature": {"mod": 2, "ability": "INT", "proficient": false, "expertise": false},
      "perception": {"mod": 4, "ability": "WIS", "proficient": false, "expertise": false},
      "performance": {"mod": 13, "ability": "CHA", "proficient": true, "expertise": false},
      "persuasion": {"mod": 9, "ability": "CHA", "proficient": true, "expertise": false},
      "religion": {"mod": 2, "ability": "INT", "proficient": false, "expertise": false},
      "sleight_of_hand": {"mod": 2, "ability": "DEX", "proficient": false, "expertise": false},
      "stealth": {"mod": 2, "ability": "DEX", "proficient": false, "expertise": false},
      "survival": {"mod": 2, "ability": "WIS", "proficient": false, "expertise": false}
  },
  "proficiencies": {
    "armor": ["Light", "Medium", "Heavy", "Shields"],
    "weapons": ["Simple", "Martial"],
    "masteries": ["Topple", "Sap", "Nick"],
    "tools": ["Calligrapher's Supplies"],
    "languages": ["Common", "Elvish", "Celestial"]
  },
  "spellcasting": {
    "slots": {
      "L1": { "total": 4, "expended": 0 },
      "L2": { "total": 3, "expended": 0 },
      "L3": { "total": 3, "expended": 0 },
      "L4": { "total": 3, "expended": 0 },
      "L5": { "total": 3, "expended": 1 }
    },
    "classes": [
      {
        "name": "Bard",
        "ability": "Charisma",
        "save_dc": 14,
        "attack_bonus": 6,
        "spells": [
          {
            "id": "spell-sheild",
            "name": "Shield",
            "prepared": true,
            "level": 1,
            "action": "reaction",
            "time": "1 Reaction",
            "range": "Self",
            "concentration": false,
            "ritual": false,
            "components": "V, S",
            "materials": "pebble"
          },
          {
            "id": "spell-vicious-mockery",
            "name": "Vicious Mockery",
            "prepared": true,
            "level": 0,
            "action": "action",
            "time": "1 Action",
            "range": "60 Feet",
            "concentration": false,
            "ritual": false,
            "components": "V",
            "materials": "",
            "description": "Unleash a string of insults. The target must succeed on a Wisdom save or take 1d4 Psychic damage and have Disadvantage on its next attack roll."
          }
        ]
      },
      {
        "name": "Wizard",
        "ability": "Intelligence",
        "save_dc": 19,
        "attack_bonus": 11,
        "spells": [
          {
            "id": "spell-healing-word",
            "name": "Healing Word",
            "prepared": true,
            "level": 1,
            "action": "bonus",
            "time": "1 Bonus Action",
            "range": "60 Feet",
            "concentration": false,
            "ritual": false,
            "components": "V",
            "materials": "",
            "description": "A creature of your choice that you can see regains Hit Points equal to 1d4 + your Spellcasting Ability modifier."
          },
          {
            "id": "spell-detect-magic",
            "name": "Detect Magic",
            "prepared": true,
            "level": 1,
            "action": "action",
            "time": "1 Action",
            "range": "Self",
            "concentration": true,
            "ritual": true,
            "components": "V, S",
            "materials": "",
            "description": "For the duration, you sense the presence of magic within 30 feet of you. You can use your action to see a faint aura around visible creatures or objects that bear magic."
          }
        ]
      }
    ]
  },
  "inventory": {
    "currency": { "cp": 1, "sp": 0, "ep": 0, "gp": 500, "pp": 0, "value": 500.01},
    "weight": 80,
    "value": 1024,
    "weight_capacity": 120,
    "attunement_slots": { "max": 3, "used": 1 },
    "items": [
      {
        "id": "item_weapon_001",
        "name": "Longsword",
        "type": "weapon",
        "weight": 3,
        "rarity": "Common",
        "value": 10,
        "meta": {
          "equipped": true,
          "damage": "1d8",
          "damage_type": "Slashing",
          "properties": ["Versatile (1d10)"],
          "mastery_property": "Sap",
          "is_magic": false,
          "atk_bonus": 0
        },
        "description": "A classic martial weapon."
      },
      {
        "id": "item_armor_002",
        "name": "Plate Armor",
        "type": "armor",
        "weight": 65,
        "rarity": "Common",
        "cost": 150,
        "meta": {
          "equipped": true,
          "is_sheild": false,
          "ac_base": 18,
          "stealth_disadvantage": true,
          "strength_requirement": 15
        },
        "description": "Heavy interlocking metal plates."
    },
      {
        "id": "item_consumable_002",
        "name": "Potion of Healing",
        "type": "consumable",
        "action": "bonus",
        "weight": 0.5,
        "rarity": "Uncommon",
        "value": 0.5,
        "meta": { "quantity": 3, "effect": "heal", "roll": "2d4+2" },
        "description": "A bubbling red liquid."
      },
      {
        "id": "item_003",
        "name": "Wand of Magic Missiles",
        "type": "wand",
        "action": "utilize",
        "weight": 0.5,
        "rarity": "Rare",
        "value": 50,
        "magical": {"attunable": false, "attuned": false},
        "meta": { "charges": 2, "charges_max": 3, "effect": "damage", "damage_type": "magical", "roll": "3d4+4" },
        "description": "Cast three Magic Missiles up to three targets."
      },
      {
        "id": "item_004",
        "name": "Spell Scroll (Shield)",
        "type": "scroll",
        "action": "bonus",
        "equipable": false,
        "magical": {"attunable": false, "attuned": false},
        "meta": { "quantity": 1, "spell_level": 1, "consumed_on_use": true },
        "description": "One-time use Shield scroll."
      }
    ]
  },
  "traits": [
    { "id": "trait-darkvision", "name": "Darkvision", "type": "origin", "description": "60ft range." },
    { "id": "trait-fey-ancestry", "name": "Fey Ancestry", "type": "origin", "description": "Advantage vs Charmed." }
  ],
  "features": [
    {
      "id": "boon-of-spell-recall",
      "name": "Boon of Spell Recall",
      "type": "Epic Boon",
      "action": "passive",
      "description": "Level 19 feature; chance to not expend spell slots."
    },
    {
      "id": "arcane-recovery",
      "name": "Arcane Recovery",
      "type": "Class Feature",
      "action": "short rest",
      "meta": { "max": 1, "current": 1, "reset": "Long Rest" }
    }
  ],
  "feats": [
      {
        "id": "feat-sage",
        "name": "Sage",
        "type": "origin",
        "action": "passive",
        "description": "Magic Initiate (Wizard)"
      }
  ],
  "about": {
    "size": "Medium",
    "age": "27",
    "height": "5'10",
    "weight": "165 lbs",
    "appearance": "Aging wizard seeking lost knowledge.",
    "personality": "I live for the roar of the crowd and the thrill of battle. I am loud, boisterous, and fiercely protective of the weak.",
    "backstory": "Seeking the truth of a long-forgotten heresy."
  },
    "identity": {
    "archetype": "Warlord",
    "adjectives": ["strong","fearless"],
    "description": "A charismatic warlord-poet clad in gleaming half-plate, conducting battle with a blade in one hand and a horn in the other.",
    "tags": ["support", "melee", "face", "control"]
  },
  "description": "Guardian of the innocents and Bladeslinger Extrodinairre!"
}
