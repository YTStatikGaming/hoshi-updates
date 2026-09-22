# Player, subtitles and media libraries

Hoshi 0.2 uses a custom browser player and keeps your preferences on the server, separately for each account. Install maintained FFmpeg and FFprobe packages to enable all supported media features. Settings reports whether the server can find them. You may set `HOSHI_FFMPEG_PATH` and `HOSHI_FFPROBE_PATH` to their executable paths and restart Hoshi. Basic direct playback remains available without these tools.

## Choose source folders

An administrator can open **Settings → Add a library → Browse**, navigate drives/mounted folders, and choose **Use this folder**. This lists the server's filesystem, including when the browser runs on another device. It is not a browser upload or your client's local file picker. Choose movies, series, anime or music, then scan. Nested directories are included. Network storage must be mounted and readable by the server account. Overlapping libraries and symlinks inside a library are rejected.

Use `Show Name/Season 01/Show Name S01E01 - Episode title.mkv` for reliable episode grouping. Movie titles may include the year. Details can be edited without renaming or changing media files. Those edits survive scans.

Artwork comes from an adjacent image sharing the media basename, `poster`, `folder`, or `cover`, in JPG/JPEG/PNG/WebP format. Otherwise FFmpeg generates a frame-based cover. Hoshi does not invent provider metadata or download commercial posters automatically.

## Audio and subtitles

Open a title's details to see its detected languages, codecs, channels and chapters. In the player, open **Playback settings** to choose audio, captions, quality or speed. Track names reflect embedded language/title tags; missing tags are labeled unspecified rather than guessed.

Selecting a different embedded audio track starts a software H.264/AAC stream from the current position. Audio is currently mixed to two channels. Quality choices set an upper resolution; they do not upscale small sources or implement adaptive streaming. Track changes preserve the absolute media timeline, and converted caption times are rebased to match seeks. There can be a short restart while a new conversion begins.

Supported text subtitles include embedded SRT/SubRip, ASS/SSA, WebVTT and MP4 timed text, plus matching `.vtt`, `.srt`, `.ass` and `.ssa` sidecars. Example: `Episode 01.en.srt` alongside `Episode 01.mkv`. Plain captions are converted to WebVTT. Styled ASS/SSA uses FFmpeg/libass burn-in to preserve formatting. Embedded PGS and DVD/VobSub bitmap tracks are rendered into converted video. DVB/XSub and external IDX/SUB pairs remain unsupported and are labeled accordingly. Attached fonts are not extracted; ASS uses fonts available on the server.

Bitmap seeking reads subtitle packets from the original timeline so a cue that started before the requested seek point remains visible. This can take longer on large files. The original video and subtitles are never modified.

Language, subtitle enablement, speed, automatic next episode and automatic intro skipping are saved per account. FFprobe supplies track metadata; a filename extension alone does not identify codecs or languages.

## Timeline previews and controls

Hover over the timeline on a desktop, or scrub it on a touch device, to see a real image and time from the video. The first request prepares a cached sprite sheet in the background. Subsequent playback reuses it until the media file changes or the bounded cache evicts it. Previews generally sample every five seconds; very long files use a wider interval to cap processing and storage. Preparation failures are visible and do not stop playback.

Use the chapter selector to jump to labeled chapters. Next/previous episode controls follow detected series and season order. Autoplay can advance to the next episode; turn it off in playback preferences if desired.

| Key | Action |
| --- | --- |
| Space or K | Play/pause |
| J / L | Back/forward 10 seconds |
| Left / Right | Back/forward 5 seconds |
| F | Fullscreen |
| M | Mute/unmute |
| C | Toggle captions |

Shortcuts do not intercept typing into form fields. Picture-in-Picture and fullscreen depend on the browser/device.

## Intro detection and skipping

Labeled opening chapters become verified intro markers automatically. Anime and series scans also start a background analysis job. **Settings → Analyze intros** shows progress and can rerun it. Audio comparisons look for substantial repeated openings near the start of related episodes, allowing different opening offsets; an arbitrary fixed number of seconds is never used as an inferred intro.

Fingerprint matches are suggestions. Open title details, preview the suggested opening, then confirm or correct its start/end values. The **Skip intro** button appears on the right while a verified intro is playing. **Automatically skip verified intros** can be switched on or off. It skips chapter-based or administrator-confirmed markers and leaves unconfirmed guesses alone. Clearing a marker suppresses rediscovery for that unchanged file.

Repeated dialogue, recap music, different language mixes, edited openings and missing metadata can prevent a reliable match. Hoshi does not promise to detect every opening. See [analysis implementation and bounds](INTELLIGENCE.md) for the exact algorithm and resource limits.

## Resource use and upgrade

Software conversion uses CPU, with at most two active conversion streams. Preview/audio-analysis jobs run through a bounded queue and a 256 MiB analysis cache. Poster frames have a separate bounded cache. These caches live under the server data directory; source media remains read-only. Use a local disk with enough space and a server account with read access to media.

Before upgrading from 0.1, stop Hoshi and back up the complete data directory. Version 0.2 adds preference, metadata-override and intro tables; existing media, accounts, watchlists and progress are retained. Scan existing libraries to start analysis or open an individual title to read new track details.
