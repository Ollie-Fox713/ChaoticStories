# assets — Static Assets

Card images, card data JSON, and UI artwork. Served locally by nginx (docker-compose) or via CDN (`assets.portal-to-perim.com`) in production.

## Structure

```
assets/
  cards/          Card images (~1211 PNG files)
    attacks/      Attack card images
    battlegear/   Battlegear card images
    creatures/    Creature card images
    locations/    Location card images
    mugic/        Mugic card images
  data/           Card stat data (JSON, source of truth for seeding)
    attacks.json
    battlegear.json
    creatures.json
    locations.json
    mugic.json
    achievements.json  Achievement definitions (seeded into achievement_definitions table)
    location_loot.json Per-location loot table pools
    stories/        Adventure story JSON (locale-based)
      README.md     Full story JSON schema reference
      en/           English stories (10 files)
  avatar/         2D layered avatar PNGs (52 files, 512x768 transparent)
    body/         Skin tone body silhouettes
    hair/         Hair style overlays
    shirt/        Shirt overlays
    pants/        Pants overlays
    shoes/        Shoes overlays
    stickers/     Profile stickers (~128x128 transparent PNGs)
    README.md     Full asset spec, naming convention, z-order, file list
  ui_images/      UI artwork
    Map_of_the_OverWorld.png
    Map_of_the_UnderWorld.png
```

## Card Images

Image URL pattern: `/assets/cards/{category}/{CardName}.png`

The `art_link` field in the `cards` DB table stores this path. To re-download card images from the Chaotic fandom wiki:

```bash
go run ./scripts/scraper
```

**Local dev**: Assets are bind-mounted into the nginx container via `docker-compose.yml` — no rebuild needed after scraping.

**Production**: Upload to S3 with `go run ./scripts/sync_assets`, served via Cloudflare CDN at `https://assets.portal-to-perim.com`.

## Card Data JSON

Each JSON file is an array of card objects. Fields vary by card type but common fields include: `name`, `set`, `rarity`, `ability`, `special_logic` (JSONB for engine-parseable effects).

**Note:** The JSON files do NOT include a `type` field — the type is implicit from which file the card is in (e.g. cards in `creatures.json` are creatures). When loading in tests, set `cd.Type` explicitly after calling `cardDataFromRaw()`.

These JSON files are the **source of truth for tests** — all card-under-test data in `cmd/api/*_test.go` is loaded from these files via `loadCards()`/`loadCardsBySet()` and converted with `cardDataFromRaw()`.

### Card Sets (13 total)
| Code | Name | Cards |
|------|------|-------|
| DOP | Dawn of Perim | 239 |
| AU | Alliance Unraveled | 246 |
| MI | M'arrillian Invasion | 231 |
| TOTT | Turn of the Tide | 123 |
| ROTO | Rise of the Oligarch | 103 |
| FUN | Forged Unity | 101 |
| SS | Secrets of the Lost City (Silent Sands) | 100 |
| ZOTH | Zenith of the Hive | 120 |
| LR | League Rewards | 24 |
| OP1 | Organized Play 1 | 24 |
| BR | Beyond the Doors | 14 |
| FAS | Fire and Stone | 15 |
| SAS | Starter (Alliances) | 1 |
