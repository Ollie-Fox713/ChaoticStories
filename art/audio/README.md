# Audio Assets

Audio files for the Chaotic TCG music player and sound effects system.

## Directory Structure

```
audio/
├── music.json   Music track manifest
├── sfx.json     SFX manifest
├── voices.json  Voice line manifest
├── music/       Background music tracks (looped per context)
├── sfx/         Sound effects (short, fire-and-forget)
└── voices/      Character voice lines (organized by creature)
```

All three manifests are the source of truth in ChaoticStories and pulled here by the sync script.

## Format

**WebM with Opus codec** (`.webm`) — excellent compression, wide browser support.

Recommended encoding:
- Music: 128kbps Opus, stereo
- SFX: 96kbps Opus, mono
- Voice lines: 64kbps Opus, mono

## Music Manifest (`music.json`)

All music tracks are listed in `music.json`. The frontend fetches this manifest to discover available tracks. Each entry has:

```json
{
  "id": "hub",
  "title": "Dromes Hub",
  "artist": "Artist Name",
  "context": "hub",
  "file": "music/hub.webm"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier for the track |
| `title` | Yes | Display name shown in the audio picker |
| `artist` | No | Artist name (shown in picker if present) |
| `context` | Yes | Which page context this is the default for (`hub`, `battle`, `perim`, `landing`). Multiple tracks can share a context — they cycle sequentially when each track ends. The first one listed plays first. |
| `file` | Yes | Path relative to `audio/` directory |

**Context auto-switching**: When the player navigates to a new page, the matching context track auto-plays starting with the first track listed for that context. If multiple tracks share a context, they cycle sequentially — when one track ends, the next in the list plays, wrapping back to the first after the last. Users can override this by picking any track from the browse panel — the override persists until they click "Auto" to revert.

## Music Tracks

| File | Context | When |
|------|---------|------|
| `music/hub.webm` | `hub` | Dromes main hub |
| `music/battle.webm` | `battle` | Live battles and spectating |
| `music/perim.webm` | `perim` | Perim exploration |
| `music/store.webm` | `landing` | Landing, login, onboarding pages (cycles with landing track) |
| `music/landing.webm` | `landing` | Landing, login, onboarding pages (cycles with store track) |

## Sound Effects

Short clips triggered by game events and UI interactions.

| File | Trigger |
|------|---------|
| `sfx/click.webm` | Button/nav clicks (global) |
| `sfx/card_hover.webm` | Card hover interactions |
| `sfx/attack_impact.webm` | Attack played in battle |
| `sfx/mugic_cast.webm` | Mugic cast (generic fallback) |
| `sfx/mugic_{slug}.webm` | Mugic-specific cast sound (e.g., `mugic_song_of_fury.webm`). Tried first; falls back to `mugic_cast` if missing |
| `sfx/ability_activate.webm` | Ability activated |
| `sfx/creature_defeated.webm` | Creature defeated |
| `sfx/combat_start.webm` | Combat engagement begins |
| `sfx/victory_fanfare.webm` | Battle won |
| `sfx/defeat_sting.webm` | Battle lost |
| `sfx/scanquest_start.webm` | Scanquest started |
| `sfx/scanquest_complete.webm` | Scanquest reward collected |
| `sfx/location_reveal.webm` | Location revealed in battle |
| `sfx/encounter.webm` | Adventure encounter opened |
| `sfx/notification.webm` | General notification |
| `sfx/story_text.webm` | Adventure story typewriter |
| `sfx/skill_rolling.webm` | Skill check rolling |
| `sfx/skill_pass.webm` | Skill check passed |
| `sfx/skill_fail.webm` | Skill check failed |

## Voice Lines

Organized by creature name (slugified: lowercase, non-alphanumeric replaced with `_`).

```
voices/
├── chaor/
│   ├── default.webm
│   ├── encounter.webm
│   ├── combat_start.webm
│   ├── combat_win.webm
│   ├── combat_lose.webm
│   └── attack.webm
├── maxxor/
│   ├── default.webm
│   └── encounter.webm
└── ...
```

Voice lines are registered in `voices.json` with `creature`, `line`, and `file` fields. Only registered voices will play.

### Combat voice line IDs

| Line ID | Trigger |
|---------|---------|
| `combat_start` | Attacking creature speaks when combat begins |
| `combat_win` | Winning creature speaks after opponent is defeated |
| `combat_lose` | Defeated creature speaks on death |

Not every creature needs combat voice lines — missing lines are silently skipped.

## Adding New Audio

All audio is managed via JSON manifests in ChaoticStories and synced here. No code changes needed.

### Music tracks
1. Encode to `.webm` (Opus codec)
2. Place the file in `ChaoticStories/audio/music/`
3. Add an entry to `ChaoticStories/audio/music.json` with `id`, `title`, `artist`, `context`, `file`
4. Run `go run ./scripts/sync_i18n ./ChaoticStories` to pull into this project

### SFX
1. Encode to `.webm` (Opus codec)
2. Place in `ChaoticStories/audio/sfx/`
3. Add an entry to `ChaoticStories/audio/sfx.json` with `id`, `file`, `description`
4. Run `go run ./scripts/sync_i18n ./ChaoticStories` to pull into this project
5. Call `playSfx('id')` in the relevant component (only needed for new trigger points)

### Voice lines
1. Encode to `.webm` (Opus codec)
2. Place in `ChaoticStories/audio/voices/{creature_slug}/`
3. Add an entry to `ChaoticStories/audio/voices.json` with `creature`, `line`, `file`
4. Run `go run ./scripts/sync_i18n ./ChaoticStories` to pull into this project

## Deploying to Production (S3/CDN)

Audio files are uploaded to S3 alongside all other assets:

```bash
go run ./scripts/sync_assets
```

This walks the entire `assets/` directory and uploads to the `assets.portal-to-perim.com` S3 bucket (fronted by Cloudflare CDN). Audio binary files (`.webm`) get 7-day immutable cache headers. Audio JSON manifests (`music.json`, `sfx.json`, `voices.json`) get 1-hour cache headers so updates propagate quickly.

In CI, the **Sync Assets** GitHub Actions workflow runs both steps automatically:
1. `go run ./scripts/sync_i18n ./ChaoticStories` — pulls latest audio from ChaoticStories
2. `go run ./scripts/sync_assets` — uploads everything to S3

The frontend reads `VITE_ASSET_BASE_URL` at build time (set to `https://assets.portal-to-perim.com` in production). All audio URLs resolve to the CDN automatically.
