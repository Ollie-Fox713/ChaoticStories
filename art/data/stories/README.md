# Story JSON Schema

Stories are adventure narratives stored as JSON in `assets/data/stories/{locale}/`. The seed script upserts them into the `stories` DB table (PK: `id` + `locale`).

For story-writing process guidance, see [`STORY_AUTHORING_WORKFLOW.md`](./STORY_AUTHORING_WORKFLOW.md). For voice references, see [`WRITING_SAMPLES.md`](./WRITING_SAMPLES.md). For starter structures, see [`WRITER_TEMPLATES.md`](./WRITER_TEMPLATES.md).

## Full Field Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | string | **required** | Unique story identifier |
| `title` | string | **required** | Display title |
| `desc` | string | `""` | Short description shown in adventure list |
| `locations` | string[] | `[]` | Parent location names — story spawns if scanquest is at any of these |
| `tribes` | string[] | `[]` | Tribe filter — story spawns if location's tribe matches |
| `unique` | bool | `false` | If true, player can only see this story once |
| `steps` | Step[] | **required** | Interactive narrative steps |
| `isWorldEvent` | bool | `false` | Tracks global choice tallies |
| `worldEventId` | string | `""` | Key for world event tracking |
| `outcomeLabels` | object | `{}` | `{choiceKey: "Display Label"}` for world event UI |
| `rewardOverride` | DropWeights | `null` | Custom drop weight table for rewards |
| `lootTable` | LootTable | `null` | Per-category card name pools (see Loot Tables below) |
| `witnessAttacks` | string[] | `[]` | Specific attack card names to grant instead of random drops; falls through to lower-priority rewards if none are eligible |
| `aiBattle` | AIBattleConfig | `null` | Custom battle creature (`{creatureName, tribe, difficulty}`) |
| `rarity` | string | `"common"` | `"common"` / `"uncommon"` / `"rare"` / `"legendary"` — controls UI color |
| `spawnWeight` | int | `0` | Custom spawn weight. `0` = use rarity default |
| `guaranteed` | bool | `false` | Always spawns if conditions match |
| `grantAchievements` | string[] | `[]` | Achievement IDs to grant on successful completion |
| `chainId` | string | `""` | Quest chain identifier |
| `chainName` | string | `""` | Display name for the quest chain (read from step 0 for quest log) |
| `chainDesc` | string | `""` | One-line description for the quest log |
| `chainIcon` | string | `""` | Emoji/character icon for the quest log |
| `chainTotalSteps` | int | `0` | Total number of steps in the chain |
| `chainStep` | int | `0` | Step number within the chain (0 = entry point) |
| `chainNext` | string | `""` | Story ID of the next step (documentation only) |
| `chainNextLocation` | string | `""` | Location name where the next step takes place |
| `chainRequiresFlags` | string[] | `[]` | Player must have ALL these flags to see the story |
| `chainSetsFlags` | string[] | `[]` | Flags set on the player upon successful completion |

## Rarity Tiers

| Rarity | Default Weight | UI Color | Notes |
|--------|---------------|----------|-------|
| common | 100 | `#8899aa` (gray) | Default for all existing stories |
| uncommon | 40 | `#44cc44` (green) | ~2.5x rarer than common |
| rare | 10 | `#00aaff` (blue) | ~10x rarer, modal gets colored border |
| legendary | 2 | `#aa44ff` (purple) | ~50x rarer, glowing border animation + sparkles |

If `spawnWeight` is set to a non-zero value, it overrides the rarity default.

## Guaranteed Stories

When `"guaranteed": true`, the story fills adventure slots before the weighted random pool. If more guaranteed stories match than available slots, they're selected by weight among themselves.

## Quest Chains

Quest chains link multiple stories into a sequential narrative. Chain progress is visible on the player's **Profile** page under the **Quests** tab — other players can see it too. The quest log only reveals completed steps and the immediate next step; future steps remain hidden.

### How it works

1. All stories in a chain share the same `chainId`
2. `chainStep: 0` is the entry point — no prerequisites beyond normal location/tribe filtering
3. Step N requires the player to have completed step N-1 of the same chain (tracked in `player_chain_progress`)
4. `chainSetsFlags` grants named flags on completion — other stories can require these via `chainRequiresFlags`
5. `chainNext` is purely informational (for tooling/documentation)

### Chain metadata (quest log)

The quest log on the Profile page reads display metadata from the `chainStep: 0` story. These fields should be set on every story in the chain for consistency, but only step 0's values are used.

| Field | Description |
|-------|-------------|
| `chainName` | Display name shown in the quest log (e.g. `"Chaor's Hunt"`) |
| `chainDesc` | One-line description shown under the name |
| `chainIcon` | Emoji/character icon displayed next to the name |
| `chainTotalSteps` | Total steps in the chain (used internally, not shown to player) |
| `chainNextLocation` | Location name where the next step takes place — shown as a hint |

### Chain example (3-step)

```json
// Step 0 — entry point
{
  "id": "ruins_pt1",
  "title": "The Forgotten Ruins",
  "chainId": "forgotten_ruins",
  "chainName": "The Forgotten Ruins",
  "chainDesc": "Uncover what lies beneath Kiru City.",
  "chainIcon": "🏛",
  "chainTotalSteps": 3,
  "chainStep": 0,
  "chainNext": "ruins_pt2",
  "chainNextLocation": "Kiru City",
  "chainSetsFlags": ["ruins_entered"],
  "locations": ["Kiru City"],
  ...
}

// Step 1
{
  "id": "ruins_pt2",
  "title": "Deeper Into Darkness",
  "chainId": "forgotten_ruins",
  "chainTotalSteps": 3,
  "chainStep": 1,
  "chainNext": "ruins_pt3",
  "chainNextLocation": "Kiru City",
  "chainRequiresFlags": ["ruins_entered"],
  "chainSetsFlags": ["ruins_depths"],
  "locations": ["Kiru City"],
  ...
}

// Step 2 — finale
{
  "id": "ruins_pt3",
  "title": "The Ancient Guardian",
  "chainId": "forgotten_ruins",
  "chainTotalSteps": 3,
  "chainStep": 2,
  "chainNext": "",
  "chainNextLocation": "",
  "chainRequiresFlags": ["ruins_depths"],
  "grantAchievements": ["explorer"],
  "rarity": "rare",
  "locations": ["Kiru City"],
  ...
}
```

## Location / Tribe Filtering

- Both `locations` and `tribes` empty → global fallback, can spawn anywhere
- Both set → OR logic (matches either location OR tribe)
- Only `locations` set → must match location
- Only `tribes` set → must match tribe

## Achievement IDs

Achievement IDs come from `achievements.json`. The current list includes:
`first_blood`, `winning_streak_3`, `winning_streak_5`, `winning_streak_10`, `flawless_victory`, `underdog`, `comeback_king`, `speed_demon`, `marathon_match`, `total_wins_10`, `total_wins_50`, `total_wins_100`, `total_games_25`, `collector_100`, `collector_500`, `full_set`, `tribe_collector`, `deck_builder`, `ow_mastery`, `uw_mastery`, `mip_mastery`, `dan_mastery`, `codemaster_slayer`, `all_codemasters`, `ranked_veteran`, `chaor_quest`

## Step / Option Schema

```json
{
  "steps": [
    {
      "text": "Narrative text shown to the player...",
      "voiceLine": { "creature": "Chaor", "line": "quest_summons" },
      "options": [
        { "label": "Choice text", "go": 1 },
        { "label": "Skill check", "skill": "SCAN DEPTH", "pass": "reward", "fail": 1 },
        { "label": "Fight", "outcome": "battle" },
        { "label": "Claim reward", "outcome": "reward" },
        { "label": "Leave", "outcome": "fail" }
      ]
    }
  ]
}
```

### Step Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | string | **required** | Narrative text shown to the player (typewriter animation) |
| `voiceLine` | object | `null` | Optional voice line to play when this step loads |
| `voiceLine.creature` | string | — | Creature display name (e.g. `"Chaor"`) — slugified for audio lookup |
| `voiceLine.line` | string | — | Voice line ID (e.g. `"quest_summons"`) — maps to `voices/{slug}/{line}.webm` |
| `options` | Option[] | **required** | Player choices |

Options can navigate to other steps (`go: stepIndex`), trigger skill checks (`skill` + `pass`/`fail`), or end the adventure (`outcome: "reward"|"fail"|"battle"`).

### Option Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `label` | string | **required** | Choice text shown to the player |
| `go` | int | — | Navigate to this step index |
| `outcome` | string | — | `"reward"` / `"fail"` / `"battle"` — ends the adventure |
| `skill` | string | — | Skill check name (e.g. `"SCAN DEPTH"`) |
| `pass` | int\|string | — | Target on skill pass (step index or outcome) |
| `fail` | int\|string | — | Target on skill fail (step index or outcome) |
| `choiceKey` | string | `""` | Key for world event tracking |
| `setsFlags` | string[] | `[]` | Flags set on the player when this choice leads to completion (quest divergence) |
| `rewardCards` | string[] | `[]` | Card names to guarantee-grant as the reward (overrides normal loot) |
| `relationshipChanges` | RelChange[] | `[]` | Creature affinity changes applied on completion |

### Relationship Change Object

| Field | Type | Description |
|-------|------|-------------|
| `creature` | string | Creature display name (must match card name in DB) |
| `delta` | int | Affinity change — positive increases, negative decreases |

### Quest Divergence

Per-option `setsFlags` **supplement** the story-level `chainSetsFlags`. Different options can set different flags, gating different follow-up stories via `chainRequiresFlags`.

```json
{
  "text": "Chaor demands your loyalty. Van Bloot whispers from the shadows…",
  "options": [
    {
      "label": "Swear allegiance to Chaor",
      "outcome": "reward",
      "setsFlags": ["chaor_allied"],
      "rewardCards": ["Chaor"],
      "relationshipChanges": [
        { "creature": "Chaor", "delta": 15 },
        { "creature": "Van Bloot", "delta": -10 }
      ]
    },
    {
      "label": "Accept Van Bloot's deal",
      "outcome": "reward",
      "setsFlags": ["vanbloot_allied"],
      "rewardCards": ["Van Bloot"],
      "relationshipChanges": [
        { "creature": "Van Bloot", "delta": 15 },
        { "creature": "Chaor", "delta": -10 }
      ]
    }
  ]
}
```

The two follow-up stories would use:
- `"chainRequiresFlags": ["chaor_allied"]` for the Chaor path
- `"chainRequiresFlags": ["vanbloot_allied"]` for the Van Bloot path

### Per-Option Loot Override

`rewardCards` guarantees one of the named cards as the reward (picked randomly if multiple, skipping already-owned cards). If the player owns all listed cards, falls through to normal loot resolution. This takes priority over `witnessAttacks` and `lootTable`.

## AI Battle Config

```json
{
  "aiBattle": {
    "creatureName": "Chaor",
    "tribe": "UnderWorld",
    "difficulty": "hard"
  }
}
```

Difficulties: `easy` (60 energy), `medium` (80), `hard` (95).

## Reward Override

```json
{
  "rewardOverride": {
    "creature": 40,
    "mugic": 20,
    "battlegear": 15,
    "subLocation": 15,
    "topLocation": 10
  }
}
```

Valid keys: `creature`, `mugic`, `battlegear`, `subLocation`, `topLocation`. Weights control the drop category probability. Omitted fields use the default weights.

## Loot Tables

Loot tables restrict which specific cards can drop per category. When a category has a non-empty pool, only those named cards are eligible (still excluding cards the player already owns). If the player owns all cards in a pool, that category's weight is zeroed and redistributed to remaining categories.

Loot tables can be set at three levels (checked in priority order):

1. **Sub-location** — `subLoc.LootTable` in `knownSubLocs` entries (`handlers.go`) — affects scanquest rewards at that specific sub-loc
2. **Parent location** — `locationLoot` map in `handlers.go` — inherited by all sub-locations under that parent, unless overridden by #1
3. **Story** — `StoryDef.lootTable` in story JSON — affects adventure rewards for that story only

```json
{
  "lootTable": {
    "creatures": ["Maxxor", "Chaor", "Tangath Toborn", "Takinom"],
    "mugic": ["Song of Fury", "Fanfare of the Vanishing"],
    "battlegear": ["Torwegg", "Vlaric Shard"],
    "attacks": [],
    "locations": ["Kiru City", "UnderWorld City"]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `creatures` | string[] | Creature card names in the drop pool |
| `mugic` | string[] | Mugic card names in the drop pool |
| `battlegear` | string[] | Battlegear card names in the drop pool |
| `attacks` | string[] | Attack card names in the drop pool |
| `locations` | string[] | Location card names in the drop pool |

All fields are optional. Empty or omitted arrays fall back to the default tribe-based pool for that category. `rewardOverride` (drop weights) and `lootTable` (card pools) work together — weights control *how likely* each category is, loot tables control *which cards* are in each category.
