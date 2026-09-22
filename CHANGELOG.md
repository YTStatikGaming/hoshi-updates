# Changelog

## 0.2.0 — playback and library experience

- Rebuilt the media interface with a cinematic player, actual artwork, detailed metadata views, grid/list browsing, sorting and responsive layouts.
- Added a server-side folder picker and editable title/synopsis/year fields that survive scans.
- Added detailed stream/chapter detection, audio-language selection, embedded text subtitles, SRT/ASS/SSA sidecars, styled ASS rendering, and PGS/VobSub bitmap burn-in with correct seek timing.
- Added real cached thumbnail sprites for timeline hover/touch previews.
- Added opening-chapter detection, conservative cross-episode audio analysis, marker review/editing, Skip intro and optional automatic skipping of verified markers.
- Added saved language/subtitle/speed/autoplay/intro preferences, next/previous episodes, chapters, fullscreen/Picture-in-Picture and keyboard controls.
- Added safe migration from 0.1, bounded analysis jobs/cache, pinned file inputs, and regressions for cancelled reads, tail-metadata MP4 and MKV seeking.
- Docker now includes distribution FFmpeg/FFprobe and fallback fonts by default.

## 0.1.0 — first alpha

Initial self-hosted server with secure first-run ownership, administrator/viewer accounts, media catalog scanning, direct browser streaming, resumable progress, optional FFmpeg compatibility playback, sidecar VTT subtitles, a persistent personal anime tracker, import/export and update notifications.

The release is an early foundation. See the README for features intentionally left for future versions.
