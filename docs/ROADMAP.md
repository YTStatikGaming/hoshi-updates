# Alpha scope and next steps

Hoshi 0.3.0 adds automatic metadata and artwork alongside a detailed media interface, multilingual playback, subtitles, timeline previews and intro skipping to the initial server and anime tracker. It remains an alpha for personal use. Production certification, broad client compatibility, large-library performance guarantees and database downgrade support have not been established.

## Playback

Current: direct file delivery, byte-range seeking, optional FFmpeg H.264/AAC output with audio/quality selection, embedded and sidecar text captions, styled ASS rendering, chapters, thumbnail previews, intro detection/review/skip, playback preferences and a custom player.

Next: broader codec negotiation and efficient remuxing, adaptive HLS/DASH, GPU acceleration, HDR tone mapping, reliable outro/recap detection, more subtitle-format coverage and broader browser/device tests.

## Library and metadata

Current: server folder browsing, filename-derived titles, FFprobe inspection, local/generated artwork, editable title/synopsis/year, library search/sort/layouts, episode relationships, rescan and missing-item handling.

Current metadata: Kitsu, TVmaze, optional TMDB, local NFO, match review, identity locks, episode descriptions and cached artwork.

Next: richer series/season matching, scheduled scans, incremental filesystem change tracking, large-library jobs, audio tags/playlists, and metadata export. Symbolic links are deliberately unsupported in library traversal.

## Accounts and anime tracking

Current: administrator/user roles, personal playback progress, watchlist statuses, episode counts, scores, favorites, notes, and JSON import/export.

Next: per-library permissions, account recovery, audit history, permitted anime tracking integrations after terms review, and explicit account linking. No third-party anime account data is synchronized in this release.

## Hosting and clients

Current: one local server process, persistent SQLite data, browser UI, Windows/Linux/Apple Silicon packaging, a systemd example, and local Docker builds.

Next: signed installers, Windows service integration, Apple notarization, Intel Mac standalone packaging, backup/restore UI, more migration coverage, security review before wider public hosting, and TV/mobile clients. Jellyfin API compatibility, DLNA, live TV/DVR, casting protocols, and plugins would each be separate projects.

## Updates

Current: a public static manifest, one backend check per interval, validation/caching/backoff, a download notification, tested binary archives, checksums, and a private-source/public-release workflow.

Next: release signing and verification, richer migration compatibility metadata, staged release channels, artifact attestations, and additional platform smoke tests. Automatic executable replacement is not part of this alpha.
