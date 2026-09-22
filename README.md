# Hoshi releases

Hoshi is a self-hosted media server with a personal anime watchlist. This public repository contains release downloads, checksums, installation notes and the update manifest. Application source is maintained separately in a private repository.

[Download the latest release](https://github.com/YTStatikGaming/hoshi-updates/releases/latest)

## First build: 0.1.0 alpha

- Browser library for movies, series, anime and music, with media folder scanning and search.
- Authenticated streaming, byte-range seeking, resume progress and sidecar WebVTT subtitles.
- Optional FFmpeg software compatibility playback. Install FFmpeg separately; it is not included in downloads.
- Personal anime lists: planned, watching, completed, on hold and dropped; episode counts, ratings, notes, favorites, JSON export/import.
- Administrator and viewer accounts, SQLite persistence, first-run ownership token and password hashing.
- Server-side update polling every 60 seconds while online, with conditional caching, bounded requests and retry backoff.

This alpha does not yet include native TV/mobile clients, hardware transcoding, automatic metadata/poster providers, AniList/MyAnimeList synchronization, embedded subtitle extraction, live TV/DVR, DLNA/casting, automatic installation of updates or full Jellyfin feature parity.

## Run it

1. Download the archive for your operating system from Releases and verify it against its `.sha256` checksum.
2. Extract the archive. On Windows run `hoshi.exe`; on Linux or macOS run `./hoshi` in a terminal.
3. Open `http://localhost:8096` and create the first administrator using the setup token printed in that terminal.
4. Open Server settings, add an absolute path to a media folder and scan it. Anime tracking also works without media files.

The server listens on the local machine by default. For a LAN server set `HOST=0.0.0.0`; use the server's address from another device. Ubuntu Server needs no desktop. Detailed Linux service, Windows startup, macOS and Docker instructions are included in each archive. Keep media folders read-only for the service account when practical. Data is stored separately from the executable; back up the data directory with Hoshi stopped before replacing a release.

## Update notifications

Hoshi reads [`latest.json`](latest.json) once per server about every minute, then shows a notice when its version is newer than the installed version. GitHub caching, outages and rate-limit backoff can delay visibility. Browsers read the server's cached result; they do not independently poll GitHub. Notifications link to the release and never install or execute downloaded files.

Each public release is accompanied by downloadable platform archives and SHA-256 checksums. The public repository automatically refreshes the manifest when a release is published, after checking that all platform downloads and checksum files exist. This also works for releases published through GitHub's interface.

This project is independent of and not affiliated with Jellyfin. Media files are provided by the person running the server.
