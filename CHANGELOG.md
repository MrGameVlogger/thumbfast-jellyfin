# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
