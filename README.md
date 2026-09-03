# thumbfast-jellyfin

A drop-in replacement for [thumbfast](https://github.com/po5/thumbfast) that fetches pre-generated thumbnails from Jellyfin's Trickplay API instead of spawning a subprocess for local thumbnail generation.

## Features

- **Instant thumbnails** — Fetches pre-generated Trickplay tiles from Jellyfin server
- **No subprocess overhead** — No need to decode video locally for thumbnails
- **HiDPI support** — Automatically scales thumbnails for Retina/HiDPI displays
- **Aspect ratio preservation** — Maintains correct16:9 (or original) aspect ratio
- **Fallback support** — Falls back to normal thumbfast for non-Jellyfin content
- **Drop-in replacement** — Compatible with any thumbfast-compatible OSC (ModernX, uosc, etc.)

## Requirements

- [mpv](https://mpv.io/) v0.38.0 or later
- [curl](https://curl.se/) (system default at `/usr/bin/curl`)
- [ffmpeg](https://ffmpeg.org/) (Homebrew at `/opt/homebrew/bin/ffmpeg`, or symlinked to `/usr/local/bin`)
- Jellyfin server with Trickplay enabled
- Any thumbfast-compatible OSC (ModernX, uosc, etc.)

## Installation

1. Download `thumbfast-jellyfin.lua`
2. Place it in your mpv scripts directory (`~/.config/mpv/scripts/`)
3. Rename to `thumbfast.lua` (replacing the original)
4. Keep a backup of the original `thumbfast.lua`

### macOS

```bash
# Backup original
cp ~/.config/mpv/scripts/thumbfast.lua ~/.config/mpv/scripts/thumbfast-upstream.lua

# Install
cp thumbfast-jellyfin.lua ~/.config/mpv/scripts/thumbfast.lua
```

### Linux

```bash
# Backup original
cp ~/.config/mpv/scripts/thumbfast.lua ~/.config/mpv/scripts/thumbfast-upstream.lua

# Install
cp thumbfast-jellyfin.lua ~/.config/mpv/scripts/thumbfast.lua
```

### Windows

Copy `thumbfast-jellyfin.lua` to `%APPDATA%\mpv\scripts\` and rename to `thumbfast.lua`.

## Configuration

Configure via `script-opts/thumbfast.conf` as usual:

```ini
# Maximum thumbnail generation size in pixels (scaled down to fit)
# Values are scaled when hidpi is enabled
max_height=225
max_width=400

# Scale factor for thumbnail display size (requires mpv 0.38+)
scale_factor=1

# Overlay id
overlay_id=42

# Enable on network playback
network=yes

# Enable hardware decoding
hwdec=yes
```

## How It Works

1. On file load, detects Jellyfin streams by URL pattern (`/Videos/{id}/stream?api_key={key}`)
2. Fetches Trickplay metadata from the Jellyfin API
3. Downloads tile images (JPEG grids of thumbnails)
4. Converts tiles to BGRA format using ffmpeg
5. Displays thumbnails instantly from server-side data
6. Falls back to normal thumbfast subprocess for non-Jellyfin content

## Jellyfin Setup

Ensure Trickplay is enabled on your Jellyfin server:

1. Go to **Dashboard** → **Libraries**
2. Select your library
3. Enable **Trickplay**
4. Run the **Generate Trickplay Images** scheduled task

## Troubleshooting

### Thumbnails not appearing

1. Check if Trickplay is enabled on your Jellyfin server
2. Verify Trickplay images have been generated for your media
3. Check mpv log for `[thumbfast]` messages

### Wrong dimensions

The script automatically calculates dimensions based on:
- `max_width` and `max_height` from `thumbfast.conf`
- Display HiDPI scale factor
- Original Trickplay tile aspect ratio

### ffmpeg not found

Ensure ffmpeg is installed and accessible:
- macOS: `brew install ffmpeg` (installs to `/opt/homebrew/bin/ffmpeg`)
- Linux: `sudo apt install ffmpeg` or equivalent
- Windows: Download from [ffmpeg.org](https://ffmpeg.org/) and add to PATH

## License

MPL-2.0 (same as upstream thumbfast)

## Credits

- [thumbfast](https://github.com/po5/thumbfast) by po5
- Jellyfin Trickplay API integration
