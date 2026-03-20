# assets/avatar — 2D Layered Avatar Assets

Transparent PNG layers that composite into a player's avatar. Each layer is a separate file rendered on top of one another via CSS absolute positioning.

## Spec

- **Dimensions**: 512 x 768 px
- **Format**: PNG with transparent background
- **Registration**: All layers must align to the same body template — identical canvas size, matching silhouette origin

## Directory Structure

```
avatar/
  body/      {body}_{pose}_{skinTone}.png
  hair/      {body}_{pose}_{style}.png
  shirt/     {body}_{pose}_{style}.png
  pants/     {body}_{pose}_{style}.png
  shoes/     {body}_{pose}_{style}.png
```

## Naming Convention

All files follow `{body}_{pose}_{variant}.png`:

- **body**: `male` | `female`
- **pose**: `standing` | `action`
- **variant**: depends on the layer category (see table below)

## Current Options

| Layer | Variants | Files per body+pose combo | Total PNGs |
|-------|----------|---------------------------|------------|
| Body (skin tone) | `light`, `medium`, `tan`, `dark`, `deep` | 5 | 20 |
| Hair | `short`, `long` | 2 | 8 |
| Shirt | `tunic`, `vest` | 2 | 8 |
| Pants | `cargo`, `shorts` | 2 | 8 |
| Shoes | `boots`, `sandals` | 2 | 8 |
| **Total** | | | **52** |

## Rendering Z-Order (bottom to top)

1. **Body** (skin tone variant)
2. **Pants**
3. **Shoes**
4. **Shirt**
5. **Hair**

The frontend `AvatarRenderer.svelte` stacks these as absolutely-positioned `<img>` elements in this order.

## Full File List

### `body/` (20 files)

```
male_standing_light.png     female_standing_light.png
male_standing_medium.png    female_standing_medium.png
male_standing_tan.png       female_standing_tan.png
male_standing_dark.png      female_standing_dark.png
male_standing_deep.png      female_standing_deep.png
male_action_light.png       female_action_light.png
male_action_medium.png      female_action_medium.png
male_action_tan.png         female_action_tan.png
male_action_dark.png        female_action_dark.png
male_action_deep.png        female_action_deep.png
```

### `hair/`, `shirt/`, `pants/`, `shoes/` (8 files each)

Pattern: `{male,female}_{standing,action}_{variant}.png`

- **hair**: `short`, `long`
- **shirt**: `tunic`, `vest`
- **pants**: `cargo`, `shorts`
- **shoes**: `boots`, `sandals`

## Adding New Options

1. Create the PNG files for all body + pose combinations (e.g., a new hair style `mohawk` needs 4 files: `male_standing_mohawk.png`, `male_action_mohawk.png`, `female_standing_mohawk.png`, `female_action_mohawk.png`)
2. Add the option to the hardcoded `avatarOptions` struct in `cmd/api/avatar.go`
3. Drop the files into the appropriate `assets/avatar/{layer}/` folder

No database changes, seed script changes, or frontend changes are needed — the API returns available options and the frontend builds URLs dynamically.

## Serving

- **Local dev** (docker-compose): Served by nginx from the bind-mounted `./assets` volume at `/assets/avatar/`. The nginx location block applies 7-day immutable caching.
- **Production** (S3 + CDN): `go run ./scripts/sync_assets` walks `assets/` recursively and uploads everything — avatar PNGs are auto-detected and cached. Served via `assets.portal-to-perim.com`.

## Art Replacement

The current files are simple colored-rectangle placeholders. To replace with real art:

1. Create new PNGs at the same 512x768 dimensions with transparent backgrounds
2. Ensure all layers for the same body+pose combination align to the same body template
3. Drop them into the same paths — same filenames, same folders
4. For local dev, no rebuild needed (bind-mounted volume). For production, run `go run ./scripts/sync_assets`
