# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.1] - 2026-11-17

### Changed
- Prose pass over README, CHANGELOG, CONTRIBUTING, AGENTS.md and the script
  header: plain punctuation instead of em dashes, sentence-case headings, and
  feature bullets that state the mechanism instead of restating their label.
  The script change is comments only
- README dedupe: one "Jellyfin clients" section replaces the "Designed for"
  lists, the duplicate OSC and use-case lists are gone, and jf-mpv-osc is
  named consistently

### Fixed
- `thumbfast.conf` comment typo: `for16:9` now reads `for 16:9`
- The README platform badge linked to an empty URL

## [1.2.0] - 2026-11-17

### Added
- OSC Preview API support (`user-data/osc/draw-preview`, mpv 0.42+): the stock
  mpv OSC now shows seekbar previews without any fork. Both the Trickplay path
  and the local fallback honor the OSC-requested display size, and the OSC's
  optional ASS decoration (thumbnail border) is rendered. The classic
  thumbfast script-message protocol keeps working unchanged for uosc,
  trickplay-jf-osc and friends
- `tls_verify` and `cacert` options for self-signed HTTPS Jellyfin servers
  (applies to curl requests and the fallback thumbnail worker, which runs
  `--no-config` and therefore never sees mpv.conf TLS settings)

### Changed
- Decode worker now uses `--load-console` / `--hwdec-software-fallback` when
  the worker mpv is 0.40+ (version-probed at startup). The pre-0.40 names are
  kept for older workers; upstream renamed both options in mpv 0.40 and the
  deprecated aliases will eventually be removed
- mpv 0.42 documentation notes: stock osc.lua support, the `--tls-verify`
  default change for stream playback, and `lua-settings/` no longer being read

## [1.1.4] - 2026-10-18

### Fixed
- Crash on successful Trickplay load: `attempt to call upvalue 'info' (a table value)`. The v1.1.2 dims re-report called `info(w, h)` from scopes where the Trickplay metadata table shadowed that function (regression in 1.1.2)

## [1.1.3] - 2026-10-16

### Fixed
- Jellyfin 12 support: the Trickplay dictionary is now `{itemId: {width: info}}`; older servers send `{width: [info]}` and both parse. Previously Jellyfin 12's GUID keys failed the width parse and every item reported "No Trickplay data"

## [1.1.2] - 2026-10-16

### Fixed
- A cancelled tile load could leak its temporary BGRA strip and file handle in the narrow window where the download callback lands after a file change (all cancel paths now discard idempotently)
- Cache hits left the internal loading flag set
- Thumbnail dimensions were never re-reported when Trickplay took over the overlay (submenus sized themselves from the fallback dimensions); also fixes a nil-global call that guard introduced
- `spawn_first` could start the decode worker while a Trickplay load was still pending
- Degenerate Trickplay dimensions are rejected instead of dividing by zero; item IDs accept dashed/uppercase GUIDs
- Removed writes to shadowed globals (`spawned`/`spawn_waiting`) in the Trickplay path

## [1.1.1] - 2026-10-16

### Fixed
- The Jellyfin API token no longer appears in the curl process list: the auth header travels via curl's stdin config (`-K -`) instead of `-H` in argv
- Overlay is removed before cache files are unlinked on file change/shutdown (never unlink a file mpv may still have mapped)
- Worker command values are quoted (edition names containing spaces broke `set edition …`)
- Version probe for pre-0.33 mpv restored (was hardcoded); duplicate `os_name` declaration removed

## [1.1.0] - 2026-10-16

### Fixed
- Trickplay on root-hosted servers: the server base URL included `/Videos`, so every API call 404'd and the script silently fell back to the decode worker (subpath hosting was unaffected)
- User lookup now uses `/Users/Me`, with `/Users` as a fallback, and works on non-admin and multi-user servers
- curl calls carry connect/total timeouts and can no longer hang the player
- `os.rename` failure while caching no longer points the overlay at a file that was never written
- Corrupt or partial cached strips are rejected by frame-count validation instead of being handed to mpv's mmap (SIGBUS risk)
- Empty OSC coordinates no longer raise `argument is not a string` in `thumb()`
- Cache directory creation is platform-aware (`mkdir -p` is not available in cmd.exe)
- Trickplay width selection is deterministic (smallest width covering `max_width`)

### Changed
- Tile download/conversion is asynchronous with a process-global load generation. First-time loads no longer block the player
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
