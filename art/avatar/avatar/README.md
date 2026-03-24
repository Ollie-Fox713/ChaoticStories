# assets/avatar — 2D Layered Avatar Assets

Transparent PNG layers that composite into a player's avatar. Each layer is a separate file rendered on top of one another via CSS absolute positioning.

## Spec

- **Dimensions**: 512 x 768 px
- **Format**: PNG with transparent background
- **Registration**: All layers must align to the same body template — identical canvas size, matching silhouette origin

## Directory Structure

```
avatar/
  head/                    {variant}.png    — greyscale, tinted with player's skin color
  arms/                    {variant}.png    — greyscale, tinted with player's skin color
  hair/                    {variant}.png
  shirt/                   {variant}.png
  pants/                   {variant}.png
  avatar_backgrounds/      bg_avatar_{name}.png     — shown behind avatar layers
  battle_backgrounds/      bg_battle_{name}.png     — shown on player's half of battle board
  card_backs/              cardback_{name}.png       — custom card back art for face-down attack cards
```

## Naming Convention

All wearable files follow `{variant}.png` (e.g., `short.png`, `tunic.png`, `cargo.png`).

## Rendering Z-Order (bottom to top)

0. **Avatar Background** (optional, from `avatar_backgrounds/`)
1. **Head** (greyscale, tinted with skin color via CSS multiply blend)
2. **Arms** (greyscale, tinted with skin color via CSS multiply blend)
3. **Pants**
4. **Shirt**
5. **Hair**

The frontend `AvatarRenderer.svelte` stacks these in order. Regular layers (pants, shirt, hair) use `<img>` elements. Greyscale layers (head, arms) are rendered via `<canvas>` — the greyscale PNG is drawn first, then the player's skin color is composited on top using `source-atop` so only opaque pixels are tinted. This prevents color bleed onto transparent areas or backgrounds.

## Skin Color System

Players choose any hex color for their skin tone via a color picker, plus an **opacity slider** (0–1, default 0.7) that controls how strongly the color is applied. The head and arms layers must be **greyscale** PNGs — the canvas compositor tints them with the chosen color and opacity while preserving shading and highlights. Non-tinted layers (hair, shirt, pants) should have fully opaque pixels where the clothing art is, otherwise backgrounds will bleed through.

### AvatarConfig (stored as JSONB)

```json
{
  "skinTone": "#D4A574",
  "skinOpacity": 0.7,
  "head": "smile",
  "arms": "side",
  "hair": "bedhead",
  "shirt": "blue_striped",
  "pants": "cargo"
}
```

## Adding New Options

### Avatar wearables (head, arms, hair, shirt, pants)

1. Create the PNG file (512x768, transparent background) named `{variant}.png`
2. For head/arms: make it greyscale so the skin color tinting works correctly
3. Add an entry to `assets/data/store_items.json` with `avatar_field`/`avatar_value` set appropriately
4. Drop the file into `assets/avatar/{layer}/`
5. Re-seed (`go run ./scripts/seed`)

### Backgrounds

1. Add the PNG to `avatar_backgrounds/` or `battle_backgrounds/`
2. Add an entry to `assets/data/store_items.json` with category `avatar_background` or `battle_background`
3. Re-seed

### Card Backs

1. Add the PNG (5:7 aspect ratio) named `cardback_{name}.png` to `card_backs/`
2. Add an entry to `assets/data/store_items.json` with category `card_back`
3. Re-seed

## Serving

- **Local dev** (docker-compose): Served by nginx from the bind-mounted `./assets` volume at `/assets/avatar/`. The nginx location block applies 7-day immutable caching.
- **Production** (S3 + CDN): `go run ./scripts/sync_assets` walks `assets/` recursively and uploads everything — avatar PNGs are auto-detected and cached. Served via `assets.portal-to-perim.com`.

## Background Assets

### Avatar Backgrounds (`avatar_backgrounds/`)

Displayed behind the avatar layers in `AvatarRenderer.svelte`. Managed as store items (category `avatar_background`).

### Battle Backgrounds (`battle_backgrounds/`)

Displayed on each player's half of the battle board in `BattleGame.svelte`. Each player's background covers their side (left = player, right = opponent mirrored on x-axis). Managed as store items (category `battle_background`).

### Card Backs (`card_backs/`)

Custom card back art displayed on face-down attack cards in battle. Each player's card back is visible to their opponent. 5:7 aspect ratio (matching card dimensions). Managed as store items (category `card_back`).

All cosmetic assets (avatar backgrounds, battle backgrounds, card backs) are selectable from the Avatar Creator editor and the Store try-on preview. To add a new cosmetic: add the PNG to the appropriate folder, add an entry to `assets/data/store_items.json`, and re-seed.
