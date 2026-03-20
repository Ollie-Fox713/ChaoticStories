# Art Assets

Place art files here to sync them into the main Chaotic project.

## Directory Structure

```
art/
  cards/
    creatures/     Card art for creatures (PNG, 5:7 portrait ratio)
    attacks/       Card art for attacks (PNG, 5:7 portrait ratio)
    battlegear/    Card art for battlegear (PNG, 5:7 portrait ratio)
    mugic/         Card art for mugic (PNG, 5:7 portrait ratio)
    locations/     Card art for locations (PNG, 7:5 landscape ratio)
  ui_images/       UI art (map images, backgrounds, etc.)
  avatar/
    body/          Avatar body layer PNGs (512x768 transparent)
    hair/          Avatar hair layer PNGs
    shirt/         Avatar shirt layer PNGs
    pants/         Avatar pants layer PNGs
    shoes/         Avatar shoes layer PNGs
```

## File Naming

- **Card art**: Use the exact card name as the filename (e.g., `Maxxor.png`, `Song of Fury.png`)
- **Avatar layers**: Use the format `{body}_{pose}_{variant}.png` (e.g., `male_idle_brown.png`)
- **UI images**: Use descriptive names matching existing conventions

## Notes

- Files placed here will overwrite existing art in the main project when synced
- All card types except locations use **5:7** aspect ratio (portrait)
- Locations use **7:5** aspect ratio (landscape)
- Avatar PNGs should be 512x768 with transparent backgrounds
- After syncing, art must be uploaded to S3 via `go run ./scripts/sync_assets`
