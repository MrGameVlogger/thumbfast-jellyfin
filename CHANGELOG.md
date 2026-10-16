# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-10-16

### Fixed
- Trickplay on root-hosted servers: the server base URL included `/Videos`, so every API call 404'd and the script silently fell back to the decode worker (subpath hosting was unaffected)
- User lookup now uses `/Users/Me`, with `/Users` as a fallback — works for non-admin and multi-user servers
- curl calls carry connect/total timeouts and can no longer hang the player
- `os.rename` failure while caching no longer points the overlay at a file that was never written
- Corrupt or partial cached strips are rejected by frame-count validation instead of being handed to mpv's mmap (SIGBUS risk)
- Empty OSC coordinates no longer raise `argument is not a string` in `thumb()`
- Cache directory creation is platform-aware (`mkdir -p` is not available in cmd.exe)
- Trickplay width selection is deterministic (smallest width covering `max_width`)

### Changed
- Tile download/conversion is asynchronous with a process-global load generation — first-time loads no longer block the player
- Cache is bounded by `cache_max_mb` (default 2048 MB), oldest entries evicted first

## [1.0.0] - 2026-09-03

### Added
- Jellyfin Trickplay support for server-side thumbnails
- Automatic detection of Jellyfin streams by URL pattern
- HiDPI/Retina display support with correct aspect ratio
- Fallback to normal thumbfast for non-Jellyfin content
- Cross-platform support (Linux, macOS, Windows)
- Configuration via `script-opts/thumbfast.conf`

### Changed
- Uses `ffmpeg` from PATH instead of hardcoded paths
- Thumbnails scaled to maintain original aspect ratio

### Notes
- Based on [thumbfast](https://github.com/po5/thumbfast) by po5
- Requires Jellyfin server with Trickplay enabled
- Requires `curl` and `ffmpeg` in PATH
