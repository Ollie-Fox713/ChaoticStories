# Story Authoring Workflow

Use this workflow when asked to create or revise a story in `assets/data/stories/{locale}/`.

This document is for story-generation work. It ties together location research, era/lore constraints, voice, and JSON structure so story writing follows a repeatable process instead of relying on memory.

## Source Order

Read sources in this order before writing:

1. `docs/locations/<location>.md`
2. `docs/STORY_BACKGROUND.md`
3. `assets/data/stories/WRITING_SAMPLES.md`
4. `assets/data/stories/WRITER_TEMPLATES.md`
5. `assets/data/stories/README.md`

If the request names a specific location, start with the matching file in `docs/locations/`.

## What To Pull From Each Source

### 1. Location doc

Use the location doc to extract:

- exact location name to use in JSON
- tribe and political ownership
- geography and physical layout
- daily life, labor, hazards, infrastructure, and routines
- strategic significance
- notable inhabitants who might plausibly appear or be referenced
- existing story references so the new story does not feel redundant

Do not blindly import every fact from the location doc into the story. Use it to ground the setting.

### 2. Story background

Use `docs/STORY_BACKGROUND.md` as the lore and era filter.

This is the authority for:

- DOP / ZOTH / SS-era tone and geopolitics
- what each tribe wants
- which secrets should stay gradual
- how to handle the Cothica
- how to handle Deepmines / fringe foreshadowing
- the rule that M'arrillians do not appear directly in this era
- the player's role as a scanner, not a commander

Important: if a location doc mentions later-era or broader-setting details, filter them through `docs/STORY_BACKGROUND.md` before using them in a story for the current story set.

### 3. Writing samples

Use `assets/data/stories/WRITING_SAMPLES.md` for voice and tone calibration.

Match these house traits:

- second-person narration
- concrete, physical description
- grounded stakes before epic stakes
- humor kept dry and situational
- tribal and regional specificity
- wonder tied to labor, culture, or environment rather than generic fantasy prose

### 4. Writer templates

Use `assets/data/stories/WRITER_TEMPLATES.md` to choose the right shape:

- daily story
- linear chain story
- branching chain story

Decide the structure before drafting prose.

### 5. Schema reference

Use `assets/data/stories/README.md` to confirm:

- allowed top-level fields
- step and option structure
- quest chain mechanics
- reward fields
- relationship and flag behavior
- rarity / guaranteed / spawn behavior

This is the final structure check before writing the JSON file.

## Drafting Process

Follow this sequence:

1. Identify the location and open the matching `docs/locations/*.md` file.
2. Write down the location's physical hooks, social hooks, hazards, and routines.
3. Check `docs/STORY_BACKGROUND.md` for tribe politics, era limits, and named-character motivations.
4. Decide whether the request should become:
   - a daily story
   - a chain entry
   - a branching choice point
5. Pick a tone reference from `WRITING_SAMPLES.md`.
6. Draft the opening paragraph in the house voice before filling in JSON.
7. Build the full story in the template shape from `WRITER_TEMPLATES.md`.
8. Validate all fields against `assets/data/stories/README.md`.

## Writing Rules

### Ground the story in place

Every story should feel like it could only happen in that location.

That means using:

- location-specific labor
- local infrastructure
- tribe-specific social behavior
- hazards that belong to the terrain or political situation
- consequences that matter to the local population

Avoid generic fantasy tasks that could happen anywhere.

### Prefer local stakes

Most daily stories should be about:

- maintenance
- logistics
- patrols
- supply problems
- rituals
- local mysteries
- public events
- repair work
- small social conflicts

Epic stakes are appropriate for chain stories, but even then the scene should begin with a concrete immediate problem.

### Compose 5-story batches intentionally

When writing 5 stories for the same location, do not make all 5 purely procedural.

Each 5-story batch should include:

- at least 1 explicit wonder / immersion story
- several grounded workaday or procedural stories
- some tonal variation across seriousness, levity, tension, or ritual where the location supports it

The wonder story should:

- pause long enough to really take in the location
- use sensory detail tied to that exact place
- emphasize why creatures of that tribe care about living there
- still stay grounded in Perim rather than drifting into generic fantasy prose

The wonder story does not need to be plotless. It can still involve a task, but the main value of the scene should be awe, reverence, strangeness, beauty, or environmental grandeur specific to the location.

### Use characters carefully

Named canon characters should appear only when the location, stakes, and era make them plausible.

When using a named character:

- match their voice and motives to `docs/STORY_BACKGROUND.md`
- keep their presence meaningful
- avoid making every story a leadership scene

Many good stories should center on workers, guards, scouts, attendants, minor officials, or local specialists.

### Keep mystery deniable

For Deepmines or fringe unease:

- imply, do not explain
- let characters misattribute the phenomenon
- prefer one disturbing detail over a full explanation

No direct M'arrillian reveal in this era.

### Respect the scanner perspective

The player is a human scanner moving through Perim.

The player can:

- observe
- assist
- investigate
- earn trust
- make local choices

The player should not:

- command armies
- resolve the entire geopolitical conflict
- understand secrets too quickly

The player is also a normal human unless the story explicitly provides gear, transport, environmental assistance, or creature help.

That means the player should not casually do creature-native things such as:

- turn invisible like a Mipedian
- fly, glide, or hover unaided
- cast Mugic or use innate elemental powers
- breathe underwater for long periods
- match creature-level strength, speed, durability, or senses

If a scene needs access to a non-human capability, provide a concrete reason inside the story:

- battlegear
- a tool or vehicle
- a guided ritual with clear limits
- direct creature assistance
- a fixed environmental mechanism

Do not imply that the player simply shares the abilities of the creatures around them.

## Story Type Defaults

### Daily stories

Default assumptions:

- `unique: false`
- `rarity: "common"`
- short arc
- one local problem
- modest but flavorful reward

### Chain stories

Default assumptions:

- `unique: true`
- often `guaranteed: true`
- stronger plot progression
- more direct ties to named characters, politics, or mysteries

### Branching chain stories

Use when:

- a choice should meaningfully align the player with a faction or character
- later stories should depend on a selected path
- `setsFlags`, `chainRequiresFlags`, `rewardCards`, or `relationshipChanges` matter

## Reward Guidance

Before choosing rewards, sanity-check them against the location and tribe.

Use:

- `witnessAttacks` for simple themed rewards
- `lootTable` for tighter control of reward flavor
- `rewardCards` only when a specific choice should guarantee a specific card

Prefer rewards that feel native to the location, tribe, or event.

## Final Checklist

Before saving a story file, confirm:

- the location name in `locations` exactly matches the real location
- the story could plausibly happen in that place
- the era and lore fit `docs/STORY_BACKGROUND.md`
- the tone matches `WRITING_SAMPLES.md`
- the shape matches `WRITER_TEMPLATES.md`
- the fields validate against `assets/data/stories/README.md`
- if the story is part of a 5-story location batch, the batch includes at least 1 explicit wonder / immersion story
- the stakes are concrete
- the prose is second-person and physically grounded
- the player is acting like a normal human unless the story explicitly establishes otherwise
- the story does not resolve mysteries that should remain unresolved
- rewards and outcomes fit the story

## If Asked To "Write A Story For X"

Use this shortcut:

1. Open `docs/locations/X.md`.
2. Extract 3-5 specific hooks from that location.
3. Filter the concept through `docs/STORY_BACKGROUND.md`.
4. Pick a tone target from `WRITING_SAMPLES.md`.
5. Choose the correct JSON shape from `WRITER_TEMPLATES.md`.
6. Validate against `assets/data/stories/README.md`.
7. Then write the final story JSON.
