# thumbfast-jellyfin

A drop-in replacement for [thumbfast](https://github.com/po5/thumbfast) that fetches pre-generated thumbnails from Jellyfin's Trickplay API instead of spawning a subprocess for local thumbnail generation.

**Fork of [po5/thumbfast](https://github.com/po5/thumbfast)** — adds Jellyfin Trickplay support while maintaining full compatibility with the original script.

## Features

- **Instant thumbnails** — Fetches pre-generated Trickplay tiles from Jellyfin server
- **No subprocess overhead** — No need to decode video locally for thumbnails
- **HiDPI support** — Automatically scales thumbnails for Retina/HiDPI displays
- **Aspect ratio preservation** — Maintains correct 16:9 (or original) aspect ratio
- **Fallback support** — Falls back to normal thumbfast for non-Jellyfin content
- **Drop-in replacement** — Compatible with any thumbfast-compatible OSC

## Requirements

- [mpv](https://mpv.io/) v0.38.0 or later
- [curl](https://curl.se/) (usually pre-installed on most systems)
- [ffmpeg](https://ffmpeg.org/) (must be in PATH)
- Jellyfin server with Trickplay enabled
- Any thumbfast-compatible OSC (see UI support below)

## Installation

### 1. Backup original thumbfast

```bash
cp ~/.config/mpv/scripts/thumbfast.lua ~/.config/mpv/scripts/thumbfast-upstream.lua
```

### 2. Install thumbfast-jellyfin

Download `thumbfast-jellyfin.lua` and place it in your mpv scripts directory as `thumbfast.lua`.

**Default locations:**
- Linux/macOS: `~/.config/mpv/scripts/`
- Windows: `%APPDATA%\mpv\scripts\`

### 3. Configure (optional)

Copy `thumbfast.conf` to your mpv script-opts directory.

**Default locations:**
- Linux/macOS: `~/.config/mpv/script-opts/`
- Windows: `%APPDATA%\mpv\script-opts\`

## UI Support

thumbfast-jellyfin is compatible with any OSC that supports the original thumbfast:

- [uosc](https://github.com/tomasklaen/uosc)
- [ModernX](https://github.com/cyl0/ModernX)
- [progressbar](https://github.com/torque/mpv-progressbar)
- [tethys](https://github.com/Zren/mpv-osc-tethys)
- [modern](https://github.com/maoiscat/mpv-osc-modern/tree/with.thumbfast)
- [oscc](https://github.com/longtermfree/oscc)
- [mfpbar](https://codeberg.org/NRK/mpv-toolbox/src/branch/master/mfpbar)
- [osc.lua](https://github.com/po5/thumbfast/blob/vanilla-osc/player/lua/osc.lua) (vanilla UI fork)

For the vanilla mpv UI, use the [osc.lua fork](https://github.com/po5/thumbfast/blob/vanilla-osc/player/lua/osc.lua) from the original thumbfast repo.

## mpv Frontends

[ImPlay](https://tsl0922.github.io/ImPlay/) is auto-detected, but if you encounter issues set `mpv_path=ImPlay` in `script-opts/thumbfast.conf`.

[mpv.net](https://github.com/mpvnet-player/mpv.net) is directly supported since v7, no special configuration is required.

Other frontends and older versions of mpv.net will need [standalone mpv](https://mpv.io/installation/) accessible within PATH. The easiest way is to copy standalone mpv files inside of your frontend's installation folder.

## macOS Notes

If your mpv install is an app bundle (e.g. stolendata builds), the script will work but you may notice the Dock shakes when generating the first thumbnail. To fix this, make sure the app is in your Applications folder, then run:

```bash
sudo ln -s /Applications/mpv.app/Contents/MacOS/mpv /usr/local/mpv
```

If you installed mpv via [Homebrew](https://brew.sh/), there are no issues.

## Configuration

Configure via `script-opts/thumbfast.conf`:

```ini
# Socket path (leave empty for auto)
socket=

# Thumbnail path (leave empty for auto)
thumbnail=

# Maximum thumbnail generation size in pixels (scaled down to fit)
# Values are scaled when hidpi is enabled
max_height=225
max_width=400

# Scale factor for thumbnail display size (requires mpv 0.38+)
# Note that this is lower quality than increasing max_height and max_width
scale_factor=1

# Apply tone-mapping, no to disable
tone_mapping=auto

# Overlay id
overlay_id=42

# Spawn thumbnailer on file load for faster initial thumbnails
spawn_first=yes

# Close thumbnailer process after an inactivity period in seconds, 0 to disable
quit_after_inactivity=0

# Enable on network playback
network=yes

# Enable on audio playback
audio=no

# Enable hardware decoding
hwdec=yes

# Windows only: use native Windows API to write to pipe (requires LuaJIT)
direct_io=no

# Custom path to the mpv executable
mpv_path=mpv
```

### Option Details

| Option | Default | Description |
|--------|---------|-------------|
| `socket` | auto | Socket path for IPC communication |
| `thumbnail` | auto | Path for temporary thumbnail file |
| `max_height` | 225 | Maximum thumbnail height (scaled by HiDPI) |
| `max_width` | 400 | Maximum thumbnail width (scaled by HiDPI) |
| `scale_factor` | 1 | Display scale factor (lower quality than increasing max dimensions) |
| `tone_mapping` | auto | Tone-mapping for HDR content (`no` to disable) |
| `overlay_id` | 42 | Overlay ID for thumbnails |
| `spawn_first` | yes | Spawn thumbnailer on file load |
| `quit_after_inactivity` | 0 | Seconds before closing thumbnailer (0 = disabled) |
| `network` | yes | Enable for network streams |
| `audio` | no | Enable for audio files |
| `hwdec` | yes | Enable hardware decoding |
| `direct_io` | no | Windows: direct pipe writing (requires LuaJIT) |
| `mpv_path` | mpv | Path to mpv executable |

## Jellyfin Setup

### Enable Trickplay

1. Open Jellyfin Dashboard
2. Go to **Libraries** → Select your library
3. Enable **Trickplay**
4. Run **Scheduled Tasks** → **Generate Trickplay Images**

### Verify Trickplay is working

1. Open a video in Jellyfin web UI
2. Hover over the seekbar
3. You should see thumbnail previews

## How It Works

1. **Detection** — On file load, detects Jellyfin streams by URL pattern (`/Videos/{id}/stream?api_key={key}`)
2. **Metadata** — Fetches Trickplay metadata from Jellyfin API (`/Users/{id}/Items/{id}`)
3. **Download** — Downloads tile images (JPEG grids of thumbnails) from `/Videos/{id}/Trickplay/{width}/{index}.jpg`
4. **Convert** — Converts tiles to BGRA format using ffmpeg, scaling to match display dimensions
5. **Display** — Thumbnails are displayed instantly from server-side data
6. **Fallback** — For non-Jellyfin content, falls back to normal thumbfast subprocess

## Troubleshooting

### Thumbnails not appearing

1. Check if Trickplay is enabled on your Jellyfin server
2. Verify Trickplay images have been generated for your media
3. Check mpv log for `[thumbfast]` messages:
   ```bash
   grep -i "thumbfast\|trickplay" ~/.config/mpv/mpv.log
   ```

### Wrong dimensions

The script automatically calculates dimensions based on:
- `max_width` and `max_height` from `thumbfast.conf`
- Display HiDPI scale factor
- Original Trickplay tile aspect ratio

### ffmpeg not found

Ensure ffmpeg is installed and accessible in your PATH:

- **Linux:** `sudo apt install ffmpeg` or `sudo dnf install ffmpeg`
- **macOS:** `brew install ffmpeg`
- **Windows:** Download from [ffmpeg.org](https://ffmpeg.org/) and add to PATH

### curl not found

curl is included by default on most systems. If missing:

- **Linux:** `sudo apt install curl` or `sudo dnf install curl`
- **macOS:** `xcode-select --install`
- **Windows:** Included in Windows 10+

## For UI Developers

This API usage example code is [CC0 (public domain)](https://creativecommons.org/share-your-work/public-domain/cc0/).

### thumbfast state variable

Declare near the top of your script:

```lua
local thumbfast = {
    width = 0,
    height = 0,
    disabled = true,
    available = false
}
```

### State setter

Register near the end of your script:

```lua
mp.register_script_message("thumbfast-info", function(json)
    local data = utils.parse_json(json)
    if type(data) ~= "table" or not data.width or not data.height then
        msg.error("thumbfast-info: received json didn't produce a table with thumbnail information")
    else
        thumbfast = data
    end
end)
```

### Request thumbnails

When user hovers on seekbar:

```lua
if not thumbfast.disabled then
    mp.commandv("script-message-to", "thumbfast", "thumb",
        hovered_seconds,  -- time in seconds
        thumb_x,          -- x position
        thumb_y           -- y position
    )
end
```

### Clear thumbnails

When user leaves seekbar:

```lua
if thumbfast.available then
    mp.commandv("script-message-to", "thumbfast", "clear")
end
```

## Differences from Upstream

| Feature | thumbfast | thumbfast-jellyfin |
|---------|-----------|-------------------|
| Local thumbnail generation | ✅ | ✅ (fallback) |
| Jellyfin Trickplay support | ❌ | ✅ |
| HiDPI-aware dimensions | ✅ | ✅ |
| Aspect ratio preservation | N/A | ✅ |
| Network stream support | Optional (config) | Automatic |

## License

This project is licensed under the Mozilla Public License 2.0 - see the [LICENSE](LICENSE) file for details.

This is a fork of [thumbfast](https://github.com/po5/thumbfast) by [po5](https://github.com/po5), which is also licensed under MPL-2.0.

## Credits

- [thumbfast](https://github.com/po5/thumbfast) by [po5](https://github.com/po5) — Original thumbnailer script
- [Jellyfin](https://jellyfin.org/) — Trickplay API
- [mpv](https://mpv.io/) — Media player

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## Related Projects

- [thumbfast](https://github.com/po5/thumbfast) — Original thumbnailer
- [ModernX](https://github.com/cyl0/ModernX) — Modern OSC for mpv
- [uosc](https://github.com/tomasklaen/uosc) — Minimalist OSC
- [Jellyfin MPV Play](https://github.com/MrGameVlogger/Jellyfin_mpv_play) — Jellyfin client for mpv
