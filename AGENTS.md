# thumbfast-jellyfin

## Project
- Drop-in replacement for thumbfast with Jellyfin Trickplay support
- Based on thumbfast by po5: https://github.com/po5/thumbfast
- License: MPL-2.0 (same as upstream)

## Key Files
- `thumbfast-jellyfin.lua` — main script
- `thumbfast.conf` — default config for Jellyfin users
- `CHANGELOG.md` — version history
- `CONTRIBUTING.md` — contribution guidelines

## Dependencies
- `curl` — for API requests to Jellyfin server
- `ffmpeg` — for converting Trickplay tiles to BGRA format
- Both must be accessible (found at startup via common paths)

## How It Works
1. On file load, detects Jellyfin streams by URL pattern (`/Videos/{id}/stream?api_key={key}`)
2. Fetches Trickplay metadata from Jellyfin API (`/Users/{id}/Items/{id}`)
3. Downloads all tile images from `/Videos/{id}/Trickplay/{width}/{index}.jpg`
4. Concatenates into one flat BGRA file (per jf-mpv-osc README optimization)
5. Caches locally in `/tmp/thumbfast-jellyfin-cache/`
6. Displays thumbnails via offset arithmetic

## Important Notes
- Uses `run_subprocess` helper for macOS PATH compatibility (ffmpeg at `/opt/homebrew/bin/ffmpeg`)
- Cache key: `{itemId}_{scaledW}x{scaledH}_{tilesX}x{tilesY}.bgra`
- HiDPI scaling: dimensions multiplied by `display-hidpi-scale` property
- Falls back to normal thumbfast subprocess for non-Jellyfin content

## Upstream Sync
- `thumbfast-upstream.lua` in mpv config is the verified upstream copy
- Check https://github.com/po5/thumbfast for updates
- Local modifications: Trickplay support, subprocess cache increase

## Related Projects
- https://github.com/po5/thumbfast — original thumbnailer
- https://github.com/iwalton3/jf-mpv-osc — Jellyfin-styled OSC (uses thumbfast)
- https://github.com/MrGameVlogger/Jellyfin_mpv_play — Jellyfin client for mpv
