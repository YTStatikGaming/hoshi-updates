# Hoshi releases

Hoshi is a self-hosted media server with a personal anime watchlist. This public repository contains release downloads, checksums, installation notes and the update manifest. Application source is maintained separately in a private repository.

[Download the latest release](https://github.com/YTStatikGaming/hoshi-updates/releases/latest)

## 0.3.0 — automatic metadata and resilient libraries

- Folder-name lookup for anime through Kitsu and TV shows through TVmaze, without API keys. Optional TMDB supplies movie/series metadata using your own server-only token.
- Automatic descriptions, episode titles and plots, posters, backdrops, episode images, genres, ratings, air dates and source credits.
- Local NFO support, conservative matching, a review queue, manual identification, identity locks and field overrides that survive refreshes.
- Persistent metadata/artwork caches, bounded provider requests, offline fallback and an online-lookup privacy switch.
- Faster rescans for unchanged files, safe handling of unreadable subfolders, and separate same-title remakes.

[Metadata setup and matching guide](docs/METADATA.md) · [Providers and attribution](docs/PROVIDERS.md)

## Playback and library experience

- A detailed, responsive library for movies, series, anime and music, with a server folder picker, artwork, title details, editable descriptions, search, sorting and grid/list views.
- A custom player with real timeline hover/scrub previews, resume, chapters, speed, keyboard controls, fullscreen, Picture-in-Picture and next/previous episodes with optional autoplay.
- Japanese/English and other embedded audio-language selection; embedded and external text captions, styled ASS/SSA, PGS and VobSub subtitles.
- Intro detection from named chapters and repeated opening audio; review/correct inferred markers, use the right-side Skip intro button, or enable automatic skipping of verified intros.
- Per-account language, subtitle, playback-speed, autoplay and auto-skip preferences.
- FFmpeg software compatibility playback and resolution choices. Install FFmpeg and FFprobe separately for advanced features; they are not included in standalone downloads. The local Docker build includes them.
- Personal anime lists: planned, watching, completed, on hold and dropped; episode counts, ratings, notes, favorites, JSON export/import.
- Administrator and viewer accounts, SQLite persistence, first-run ownership token and password hashing.
- Server-side update polling every 60 seconds while online, with conditional caching, bounded requests and retry backoff.

This remains an alpha, with full Jellyfin parity still ahead. Native TV/mobile clients, hardware transcoding, adaptive streaming, third-party tracker synchronization, live TV/DVR, DLNA/casting and automatic update installation remain future work. DVB/XSub, external IDX/SUB pairs and embedded font extraction are not supported. Repeated-audio intro matches require review before auto-skipping; detection is not guaranteed for every file.

[Player and library guide](docs/PLAYBACK.md) · [Intro detection and preview details](docs/INTELLIGENCE.md)

## Run it

1. Download the archive for your operating system from Releases and verify it against its `.sha256` checksum.
2. Extract the archive. On Windows run `hoshi.exe`; on Linux or macOS run `./hoshi` in a terminal.
3. Open `http://localhost:8096` and create the first administrator using the setup token printed in that terminal.
4. Open Settings → Add a library → Browse, choose a folder on the server and scan it. Anime tracking also works without media files.
5. Install FFmpeg and FFprobe for language selection, subtitle conversion, timeline previews and audio analysis. On Ubuntu/Debian use `sudo apt install ffmpeg fonts-dejavu-core`; on Windows/macOS use a maintained build linked by [FFmpeg](https://ffmpeg.org/download.html), then add it to PATH or configure `HOSHI_FFMPEG_PATH` and `HOSHI_FFPROBE_PATH`.

**Upgrading from 0.1 or 0.2:** stop Hoshi, back up its complete data directory, replace the executable, and restart with the same data directory. Database migrations retain accounts, watchlists, libraries, manual descriptions and playback progress. Existing video libraries begin background metadata lookup automatically. Online lookup sends title queries and provider IDs, not media files or folder paths; disable it in Settings or start with `HOSHI_METADATA_ENABLED=0` for offline use.

The server listens on the local machine by default. For a LAN server set `HOST=0.0.0.0`; use the server's address from another device. Ubuntu Server needs no desktop. Detailed Linux service, Windows startup, macOS and Docker instructions are included in each archive. Keep media folders read-only for the service account when practical. Data is stored separately from the executable; back up the data directory with Hoshi stopped before replacing a release.

## Update notifications

Hoshi reads [`latest.json`](latest.json) once per server about every minute, then shows a notice when its version is newer than the installed version. GitHub caching, outages and rate-limit backoff can delay visibility. Browsers read the server's cached result; they do not independently poll GitHub. Notifications link to the release and never install or execute downloaded files.

Each public release is accompanied by downloadable platform archives and SHA-256 checksums. The public repository automatically refreshes the manifest when a release is published, after checking that all platform downloads and checksum files exist. This also works for releases published through GitHub's interface.

This project is independent of and not affiliated with Jellyfin. Media files are provided by the person running the server.
