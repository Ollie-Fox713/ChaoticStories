# assets/avatar — 2D Layered Avatar Assets

Transparent PNG layers that composite into a player's avatar. Each layer is a separate file rendered on top of one another via CSS absolute positioning.

## Spec

- **Dimensions**: 512 x 768 px
- **Format**: PNG with transparent background
- **Registration**: All layers must align to the same body template — identical canvas size, matching silhouette origin

## Directory Structure

```
avatar/
  body/                    {type}.png               — greyscale, tinted with player's skin color (male.png, female.png)
  face/                    {gender}_{variant}.png    — greyscale, tinted with player's skin color
  arms/                    {gender}_{variant}.png    — greyscale, tinted with player's skin color (male only; female arms baked into body)
  hair/                    {gender}_hair_{style}.png — male hairs are greyscale (tinted with hair color); female hairs are pre-colored
  shirt/                   {gender}_{variant}.png
  pants/                   {gender}_{variant}.png
  avatar_backgrounds/      bg_avatar_{name}.png      — shown behind avatar layers
  battle_backgrounds/      bg_battle_{name}.png      — shown on player's half of battle board
  card_backs/              cardback_{name}.png        — custom card back art for face-down attack cards
  stickers/                {name}.png                 — profile stickers players can place on their profile page
```

## Naming Convention

All wearable files use a gender prefix: `{gender}_{variant}.png` (e.g., `male_hair_bedhead.png`, `female_shirt_hoodie.png`). Body files use just the type: `male.png`, `female.png`.

## Body Types

Two body types: **male** and **female**. Each body type has its own set of wearable items. Items are gender-locked via the `body_filter` column in `store_items` — an item with `body_filter: "male"` only appears for male body type.

- **Male**: Separate body, face, arms, hair, shirt, pants layers
- **Female**: Body includes arms (no separate arms layer). Separate face, hair, shirt, pants layers.

## Rendering Z-Order (bottom to top)

0. **Avatar Background** (optional, from `avatar_backgrounds/`)
1. **Body** (greyscale, tinted with skin color via canvas)
2. **Face** (greyscale, tinted with skin color via canvas)
3. **Arms** (greyscale, tinted with skin color — male only; skipped for female)
4. **Pants**
5. **Shirt**
6. **Hair** (tinted with hair color via canvas)

The frontend `AvatarRenderer.svelte` stacks these in order using a unified layer stack. Greyscale layers (body, face, arms, hair) are rendered via `<canvas>` — the greyscale PNG is drawn first, then the tint color is composited on top using `source-atop` so only opaque pixels are tinted.

## Skin Color System

Players choose any hex color for their skin tone via a color picker, plus an **opacity slider** (0–1, default 0.7) that controls how strongly the color is applied. The body, face, and arms layers must be **greyscale** PNGs — the canvas compositor tints them with the chosen color and opacity while preserving shading and highlights.

## Hair Color System

Players choose any hex color for their hair via a color picker, plus an **opacity slider** (0–1, default 0.7). Male hair assets are **greyscale** (white/black) for optimal tinting. Female hair assets are pre-colored but can still be tinted. The hair layer is rendered via canvas with the chosen hair color and opacity.

### AvatarConfig (stored as JSONB)

```json
{
  "skinTone": "#D4A574",
  "skinOpacity": 0.7,
  "body": "male",
  "face": "male_smile",
  "arms": "male_arms",
  "hair": "male_hair_bedhead",
  "shirt": "male_tshirt_fire",
  "pants": "male_cargo_army",
  "hairColor": "#4A3728",
  "hairOpacity": 0.7
}
```

## Adding New Options

### Avatar wearables (body, face, arms, hair, shirt, pants)

1. Create the PNG file (512x768, transparent background) named `{gender}_{variant}.png`
2. For body/face/arms: make it greyscale so the skin color tinting works correctly
3. For male hair: make it greyscale (white/black) for hair color tinting
4. Add an entry to `assets/data/store_items.json` with `avatar_field`/`avatar_value` set appropriately, and `body_filter` set to `"male"` or `"female"`
5. Drop the file into `assets/avatar/{layer}/`
6. Re-seed (`go run ./scripts/seed`)

### Backgrounds

1. Add the PNG to `avatar_backgrounds/` or `battle_backgrounds/`
2. Add an entry to `assets/data/store_items.json` with category `avatar_background` or `battle_background`
3. Re-seed

### Card Backs

1. Add the PNG (5:7 aspect ratio) named `cardback_{name}.png` to `card_backs/`
2. Add an entry to `assets/data/store_items.json` with category `card_back`
3. Re-seed

### Stickers

Profile stickers are decorative images players purchase from the store and place on their profile page. Other players see the placed stickers when viewing the profile.

1. Create a square PNG (~128x128, transparent background) named `{name}.png`
2. Drop the file into `assets/avatar/stickers/`
3. Add an entry to `assets/data/store_items.json` with category `sticker`:
   ```json
   {"id": "sticker_{name}", "name": "Display Name", "category": "sticker", "price": 100, "description": "...", "preview_url": "avatar/stickers/{name}.png"}
   ```
4. Re-seed (`go run ./scripts/seed`)
5. Sync assets (`go run ./scripts/sync_assets`) to upload the PNG to S3/CDN

Sticker placements are stored server-side in the `player_stickers` table (x/y as percentages, plus scale and rotation). Players place stickers via the "CUSTOMIZE" button on their own profile, which opens a drag-and-drop editor. Max 10 stickers per profile.

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

### Stickers (`stickers/`)

Decorative images that players purchase from the store and place freely on their profile page. Stickers are ~128x128 transparent PNGs. Players position, scale, and rotate stickers via a drag-and-drop editor on their profile. Placements are stored in the `player_stickers` table and visible to all profile viewers. Managed as store items (category `sticker`).

All cosmetic assets (avatar backgrounds, battle backgrounds, card backs, stickers) are selectable from the Store. To add a new cosmetic: add the PNG to the appropriate folder, add an entry to `assets/data/store_items.json`, and re-seed.
