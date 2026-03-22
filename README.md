# ChaoticStories

> **New here?** Check out [HELPME.md](HELPME.md) for a step-by-step beginner's guide on how to install Git, clone this repo, add your files, and submit a pull request.

---

# Story JSON Schema

Stories are adventure narratives stored as JSON in `assets/data/stories/{locale}/`. The seed script upserts them into the `stories` DB table (PK: `id` + `locale`).

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
| `witnessAttacks` | string[] | `[]` | Specific attack card names to grant instead of random drops |
| `aiBattle` | AIBattleConfig | `null` | Custom battle creature (`{creatureName, tribe, difficulty}`) |
| `rarity` | string | `"common"` | `"common"` / `"uncommon"` / `"rare"` / `"legendary"` — controls UI color |
| `spawnWeight` | int | `0` | Custom spawn weight. `0` = use rarity default |
| `guaranteed` | bool | `false` | Always spawns if conditions match |
| `grantAchievements` | string[] | `[]` | Achievement IDs to grant on successful completion |
| `chainId` | string | `""` | Quest chain identifier |
| `chainStep` | int | `0` | Step number within the chain (0 = entry point) |
| `chainNext` | string | `""` | Story ID of the next step (documentation only) |
| `chainRequiresFlags` | string[] | `[]` | Player must have ALL these flags to see the story |
| `chainSetsFlags` | string[] | `[]` | Flags set on the player upon successful completion |

## Rarity Tiers

| Rarity | Default Weight | UI Color | Notes |
|--------|---------------|----------|-------|
| common | 100 | `#8899aa` (gray) | Default for all existing stories |
| uncommon | 40 | `#44cc44` (green) | ~2.5x rarer than common |
| rare | 10 | `#00aaff` (blue) | ~10x rarer, modal gets colored border |
| legendary | 2 | `#ff6b00` (orange) | ~50x rarer, glowing border animation |

If `spawnWeight` is set to a non-zero value, it overrides the rarity default.

## Guaranteed Stories

When `"guaranteed": true`, the story fills adventure slots before the weighted random pool. If more guaranteed stories match than available slots, they're selected by weight among themselves.

## Quest Chains

Quest chains link multiple stories into a sequential narrative.

### How it works

1. All stories in a chain share the same `chainId`
2. `chainStep: 0` is the entry point — no prerequisites beyond normal location/tribe filtering
3. Step N requires the player to have completed step N-1 of the same chain (tracked in `player_chain_progress`)
4. `chainSetsFlags` grants named flags on completion — other stories can require these via `chainRequiresFlags`
5. `chainNext` is purely informational (for tooling/documentation)

### Chain example (3-step)

```json
// Step 0 — entry point
{
  "id": "ruins_pt1",
  "title": "The Forgotten Ruins",
  "chainId": "forgotten_ruins",
  "chainStep": 0,
  "chainNext": "ruins_pt2",
  "chainSetsFlags": ["ruins_entered"],
  "locations": ["Kiru City"],
  ...
}

// Step 1
{
  "id": "ruins_pt2",
  "title": "Deeper Into Darkness",
  "chainId": "forgotten_ruins",
  "chainStep": 1,
  "chainNext": "ruins_pt3",
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
  "chainStep": 2,
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

Achievement IDs come from `achievements.go`. The current list includes:
`first_blood`, `winning_streak_3`, `winning_streak_5`, `winning_streak_10`, `flawless_victory`, `underdog`, `comeback_king`, `speed_demon`, `marathon_match`, `total_wins_10`, `total_wins_50`, `total_wins_100`, `total_games_25`, `collector_100`, `collector_500`, `full_set`, `tribe_collector`, `deck_builder`, `ow_mastery`, `uw_mastery`, `mip_mastery`, `dan_mastery`, `codemaster_slayer`, `all_codemasters`, `ranked_veteran`

## Step / Option Schema

```json
{
  "steps": [
    {
      "text": "Narrative text shown to the player...",
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

Options can navigate to other steps (`go: stepIndex`), trigger skill checks (`skill` + `pass`/`fail`), or end the adventure (`outcome: "reward"|"fail"|"battle"`).

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
    "battlegear": 20,
    "mugic": 15,
    "attack": 15,
    "location": 10
  }
}
```

Weights control the drop category probability. Omitted fields use the default weights.

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

All fields are optional. Empty or omitted arrays fall back to the default tribe-based pool for that category. `rewardOverride` (drop weights) and `loot tables` (card pools) work together — weights control *how likely* each category is, loot tables control *which cards* are in each category.

---

# Translation Files

This repo also hosts translation files that are synced into the main Chaotic project. Translators can work here without needing access to the game codebase.

## Directory Structure

```
i18n/
  en.json          # English UI strings (SOURCE OF TRUTH — do not edit here, pushed from main repo)
  pt.json          # Portuguese UI strings
  es.json          # Spanish UI strings

cards/
  pt/
    cards.json     # Portuguese card ability/flavor text
  es/
    cards.json     # Spanish card ability/flavor text

stories/
  pt/
    *.json         # Portuguese story translations
  es/
    *.json         # Spanish story translations

art/
  cards/
    creatures/     # Card art — PNG, 5:7 portrait ratio
    attacks/
    battlegear/
    mugic/
    locations/     # Card art — PNG, 7:5 landscape ratio
  ui_images/       # UI art (maps, backgrounds, etc.)
  avatar/
    body/          # Avatar layer PNGs (512x768 transparent)
    hair/
    shirt/
    pants/
    shoes/

audio/
  music.json       # Music track manifest (SOURCE OF TRUTH — edit here, pulled into main repo)
  sfx.json         # SFX manifest (SOURCE OF TRUTH — edit here, pulled into main repo)
  voices.json      # Voice line manifest (SOURCE OF TRUTH — edit here, pulled into main repo)
  music/           # Background music tracks (.webm Opus)
  sfx/             # Sound effects (.webm Opus)
  voices/          # Character voice lines, organized by creature
    chaor/
    maxxor/
    ...
```

## UI Strings (`i18n/*.json`)

Each file is a flat JSON object with dot-notation keys:

```json
{
  "landing.title": "CHAOTIC",
  "landing.enter": "ENTER THE GAME",
  "dromes.battle": "BATTLE",
  "faq.title": "FAQ",
  ...
}
```

**Rules:**
- `en.json` is read-only in this repo — it's pushed here automatically from the main project so translators can see all available keys
- To translate, open `pt.json` or `es.json` and fill in the empty `""` values with translations
- Keys with `{name}` placeholders (e.g., `"Welcome, {name}!"`) — keep the `{placeholder}` intact, just translate the surrounding text
- Don't add or remove keys — only fill in values

## Card Translations (`cards/{locale}/cards.json`)

Each file is a JSON array of card objects:

```json
[
  {
    "name": "Maxxor",
    "type": "creature",
    "ability_text": "",
    "flavor_text": ""
  },
  ...
]
```

**Rules:**
- `name` and `type` are identifiers — do NOT translate them
- Only fill in `ability_text` and `flavor_text` with translated text
- Leave entries empty (`""`) if no translation is available yet — the game falls back to English
- Card names are proper nouns and stay in English everywhere

## Story Translations (`stories/{locale}/*.json`)

Story files follow the same JSON schema as `stories/en/`. Copy the English file, keep the `id` and structural fields the same, and translate:
- `title` — story title
- `desc` — short description
- `steps[].text` — narrative text
- `steps[].options[].label` — choice button text

Do NOT change: `id`, `locations`, `tribes`, `go`, `outcome`, `skill`, `pass`, `fail`, or any other structural/logic fields.

## Art Assets (`art/`)

Drop art files into the appropriate subdirectory. See `art/README.md` for full details on file naming and dimensions.

- **Card art**: `art/cards/{type}/{CardName}.png` — use the exact card name as filename
- **UI images**: `art/ui_images/{name}.png`
- **Avatar layers**: `art/avatar/{layer}/{body}_{pose}_{variant}.png`

Art files placed here will overwrite existing art in the main project when synced, then get uploaded to the CDN.

## Audio Assets (`audio/`)

Music, sound effects, and character voice lines for the game. All audio files should use **WebM with Opus codec** (`.webm`).

### Music Tracks (`audio/music/`)

Background music that loops per game context (hub, battle, exploration, etc.).

- **Format:** `.webm` (Opus codec), stereo, ~128kbps recommended
- **Naming:** Use descriptive names matching the manifest (e.g., `hub.webm`, `battle.webm`)
- **Manifest:** `audio/music.json` is the track registry and lives here as the source of truth. When you add a new track file, also add an entry to the manifest so the game knows about it.
- **Adding tracks:** Drop the `.webm` file into `audio/music/`, then add an entry to `music.json` with `id`, `title`, `artist`, `context`, and `file` path. Both get synced to the main project.

### Sound Effects (`audio/sfx/`)

Short clips triggered by game events (attacks, victories, UI clicks, etc.).

- **Format:** `.webm` (Opus codec), mono, ~96kbps recommended
- **Manifest:** `audio/sfx.json` is the SFX registry. When you add a new SFX file, also add an entry to the manifest.
- **Adding SFX:** Drop the `.webm` file into `audio/sfx/`, then add an entry to `sfx.json` with `id`, `file`, and `description`. Current effects:

| File | Trigger |
|------|---------|
| `click.webm` | Button/nav clicks |
| `card_hover.webm` | Card hover |
| `attack_impact.webm` | Attack played |
| `mugic_cast.webm` | Mugic cast |
| `ability_activate.webm` | Ability activated |
| `creature_defeated.webm` | Creature defeated |
| `combat_start.webm` | Combat begins |
| `victory_fanfare.webm` | Battle won |
| `defeat_sting.webm` | Battle lost |
| `scanquest_start.webm` | Scanquest started |
| `scanquest_complete.webm` | Reward collected |
| `location_reveal.webm` | Location revealed |
| `encounter.webm` | Adventure encounter |
| `notification.webm` | General notification |
| `story_text.webm` | Story typewriter |
| `skill_rolling.webm` | Skill check rolling |
| `skill_pass.webm` | Skill check passed |
| `skill_fail.webm` | Skill check failed |

### Voice Lines (`audio/voices/`)

Character voice clips organized by creature name (lowercased, non-alphanumeric replaced with `_`).

- **Format:** `.webm` (Opus codec), mono, ~64kbps recommended
- **Manifest:** `audio/voices.json` is the voice line registry. When you add a new voice file, also add an entry to the manifest.
- **Adding voices:** Drop the `.webm` file into `audio/voices/{creature_slug}/`, then add an entry to `voices.json`.

**voices.json entry format:**
```json
{
  "creature": "Chaor",
  "line": "encounter",
  "file": "voices/chaor/encounter.webm"
}
```

| Field | Description |
|-------|-------------|
| `creature` | Creature name (exact casing — gets slugified automatically) |
| `line` | Line ID: `default`, `encounter`, `attack` (extensible) |
| `file` | Path relative to `audio/` directory |

Example directory structure:
```
audio/voices/
  chaor/
    default.webm
    encounter.webm
    attack.webm
  maxxor/
    default.webm
    encounter.webm
```

## Adding a New Language

1. Create `i18n/{code}.json` — copy `en.json` and replace all values with `""`
2. Create `cards/{code}/cards.json` — copy from `cards/pt/cards.json` (values are already empty)
3. Create `stories/{code}/` — copy story files from `stories/en/` and translate text fields
4. Let the main project maintainer know so they can add the locale code to the backend's `normalizeLocale()` function

## Syncing

The main project has a `scripts/sync_i18n/` Go script that:
1. Pushes `en.json` FROM the main project TO this repo (so translators see the latest keys)
2. Pulls all non-English translations, stories, art, audio files, and audio manifests (`music.json`, `sfx.json`, `voices.json`) FROM this repo INTO the main project

The CI/CD pipelines automatically clone this repo and run the sync before building, seeding, or uploading assets. Translators, artists, and audio contributors just need to push changes here.
