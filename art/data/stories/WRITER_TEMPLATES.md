# Writer Templates

These are copy-first templates for writing story JSON in `assets/data/stories/{locale}/`.

They are intentionally narrower than the full schema in [`README.md`](./README.md). The goal is to give story writers a safe starting point that matches what the server actually reads.

Use valid JSON when creating the real file. The examples below are ready to copy, then edit.

## 1. Daily Story Template

Use this for normal one-off adventures like `daily_ow_*`, `daily_uw_*`, `daily_mip_*`, and `daily_dan_*`.

```json
{
  "id": "daily_ow_26",
  "title": "Story Title",
  "desc": "One-line summary shown in the adventure list.",
  "locations": ["Kiru City"],
  "tribes": [],
  "unique": false,
  "rarity": "common",
  "steps": [
    {
      "text": "Opening scene text goes here.\n\nSet the problem, introduce the place, and give the player a choice.",
      "options": [
        { "label": "Investigate the obvious problem", "go": 1 },
        { "label": "Try a more careful approach", "go": 2 }
      ]
    },
    {
      "text": "This is the payoff for the first path.",
      "options": [
        { "label": "Finish the job", "outcome": "reward" }
      ]
    },
    {
      "text": "This is the payoff for the second path.",
      "options": [
        { "label": "Leave things in good order", "outcome": "reward" }
      ]
    }
  ],
  "witnessAttacks": ["Attack Name 1", "Attack Name 2"],
  "rewardOverride": null,
  "lootTable": null,
  "aiBattle": null,
  "grantAchievements": []
}
```

Notes:

- `locations` is usually the main driver for daily stories.
- Leave `tribes` empty unless you want tribe-based spawning.
- `witnessAttacks` is a simple way to theme rewards without building a custom loot table.
- Most daily stories should stay `common`, `unique: false`, and non-chain.
- When writing a batch of 5 stories for one location, make at least 1 of them a wonder / immersion story that really takes in the location's atmosphere, scale, beauty, or strangeness.

## 2. Linear Epic Chain Template

Use this for a multi-part quest where each story unlocks the next one in order.

Example below is a middle step in a linear chain.

```json
{
  "id": "ruins_pt2",
  "title": "Deeper Into The Ruins",
  "desc": "You return to the site with better tools and worse expectations.",
  "locations": ["Kiru City"],
  "tribes": [],
  "unique": true,
  "guaranteed": true,
  "rarity": "legendary",
  "chainId": "forgotten_ruins",
  "chainName": "The Forgotten Ruins",
  "chainDesc": "Uncover what lies beneath Kiru City.",
  "chainIcon": "R",
  "chainTotalSteps": 3,
  "chainStep": 1,
  "chainNext": "ruins_pt3",
  "chainNextLocation": "Kiru City",
  "chainRequiresFlags": ["ruins_entered"],
  "chainSetsFlags": ["ruins_depths"],
  "steps": [
    {
      "text": "You reach the lower chamber. The air is still, the stone is warm, and your scanner keeps catching echoes that should not be there.",
      "voiceLine": { "creature": "Maxxor", "line": "quest_warning" },
      "options": [
        { "label": "Take a careful scan of the chamber", "skill": "SCAN DEPTH", "pass": 1, "fail": 1 },
        { "label": "Move deeper without scanning", "go": 1 }
      ]
    },
    {
      "text": "The chamber opens into a buried hall. Something was built here on purpose, and someone does not want it found.",
      "options": [
        { "label": "Mark the route and report back", "outcome": "reward" }
      ]
    }
  ],
  "witnessAttacks": [],
  "rewardOverride": {
    "creature": 0,
    "mugic": 40,
    "battlegear": 40,
    "subLocation": 10,
    "topLocation": 10
  },
  "lootTable": {
    "creatures": [],
    "mugic": ["Mugic Card Name"],
    "battlegear": ["Battlegear Name 1", "Battlegear Name 2"],
    "attacks": [],
    "locations": []
  },
  "aiBattle": null,
  "grantAchievements": []
}
```

Notes:

- All stories in the chain share the same `chainId`.
- `chainStep: 0` is the entry story. Later steps require previous progress automatically.
- `chainRequiresFlags` is for extra gating.
- `chainSetsFlags` is for story-wide progression flags granted on successful completion.
- Keep `chainName`, `chainDesc`, `chainIcon`, and `chainTotalSteps` consistent across the chain.

## 3. Branching Chain Template

Use this when a player choice should split the quest into different follow-up stories.

This template shows the branching step itself.

```json
{
  "id": "faction_choice_pt3",
  "title": "A Line In The Sand",
  "desc": "Two factions want the same discovery, and both want your scanner.",
  "locations": ["The Storm Tunnel"],
  "tribes": [],
  "unique": true,
  "guaranteed": true,
  "rarity": "legendary",
  "chainId": "storm_tunnel",
  "chainName": "Storm Tunnel",
  "chainDesc": "Trace a dangerous signal beneath the dunes.",
  "chainIcon": "S",
  "chainTotalSteps": 5,
  "chainStep": 2,
  "chainNext": "branches after player choice",
  "chainNextLocation": "The Storm Tunnel",
  "chainRequiresFlags": ["storm_tunnel_midpoint"],
  "chainSetsFlags": ["storm_tunnel_choice_made"],
  "steps": [
    {
      "text": "The chamber is unstable, but that is not the real problem. The real problem is that both sides arrived at once, and both of them think you belong to them.",
      "options": [
        {
          "label": "Side with Faction A",
          "outcome": "reward",
          "setsFlags": ["storm_tunnel_faction_a"],
          "rewardCards": ["Faction A Leader"],
          "relationshipChanges": [
            { "creature": "Faction A Leader", "delta": 15 },
            { "creature": "Faction B Leader", "delta": -10 }
          ]
        },
        {
          "label": "Side with Faction B",
          "outcome": "reward",
          "setsFlags": ["storm_tunnel_faction_b"],
          "rewardCards": ["Faction B Leader"],
          "relationshipChanges": [
            { "creature": "Faction B Leader", "delta": 15 },
            { "creature": "Faction A Leader", "delta": -10 }
          ]
        }
      ]
    }
  ],
  "witnessAttacks": [],
  "rewardOverride": {
    "creature": 100,
    "mugic": 0,
    "battlegear": 0,
    "subLocation": 0,
    "topLocation": 0
  },
  "lootTable": {
    "creatures": ["Faction A Leader", "Faction B Leader"],
    "mugic": [],
    "battlegear": [],
    "attacks": [],
    "locations": []
  },
  "aiBattle": null,
  "grantAchievements": []
}
```

Follow-up stories would then use:

- `chainRequiresFlags: ["storm_tunnel_faction_a"]` for the Faction A branch
- `chainRequiresFlags: ["storm_tunnel_faction_b"]` for the Faction B branch

Notes:

- Per-option `setsFlags` supplements story-level `chainSetsFlags`.
- Per-option `rewardCards` has the highest reward priority on success.
- `relationshipChanges` updates the player's creature affinity values.

## Quick Rules

- `id` must be unique within the locale.
- `steps` is required, and each step needs `text` plus at least one option.
- Each option should resolve cleanly to either:
  - another step via `go`
  - a skill check via `skill` plus `pass` and `fail`
  - a terminal outcome via `outcome`
- Valid outcomes are `reward`, `fail`, and `battle`.
- The runtime reward weight keys are:
  - `creature`
  - `mugic`
  - `battlegear`
  - `subLocation`
  - `topLocation`
- `lootTable` keys are:
  - `creatures`
  - `mugic`
  - `battlegear`
  - `attacks`
  - `locations`

## Current Content Patterns

- Daily stories are usually:
  - `unique: false`
  - `rarity: "common"`
  - non-chain
- Epic chain stories are usually:
  - `unique: true`
  - `guaranteed: true`
  - `rarity: "legendary"`
  - chain-driven

For the full schema and feature reference, see [`README.md`](./README.md).
