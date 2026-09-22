# Seek previews and opening detection

`src/intelligence.mjs` supplies background JPEG sprite generation and local opening analysis. It uses Node's standard library and the configured FFmpeg/FFprobe executables. No media, audio, or fingerprints are uploaded.

## Server API

```js
import { createMediaIntelligence, detectIntroFromChapters } from './intelligence.mjs';

const intelligence = createMediaIntelligence({
  cacheDir: '/private/hoshi-cache/intelligence',
  ffmpegPath: '/usr/bin/ffmpeg',
  ffprobePath: '/usr/bin/ffprobe',
});

const manifest = await intelligence.requestTrickplay(item, library.path);
const status = await intelligence.getTrickplay(item, library.path);
const sprite = await intelligence.getSprite(item, library.path, 0);

const chapterIntro = detectIntroFromChapters(probe.chapters, probe.duration);
const result = await intelligence.detectSharedIntros([
  { id: 'episode-1', path: '/media/show/episode-1.mkv', root: '/media', duration: 1440 },
  { id: 'episode-2', path: '/media/show/episode-2.mkv', root: '/media', duration: 1430 },
], { signal: abortController.signal });

await intelligence.shutdown();
```

The application must resolve authenticated item IDs to its own catalog records and configured library roots. Never accept a media path, library root, executable path, or cache directory from a preview request. Every input is validated through `resolveMediaPath` and pinned by `openMediaInput`. FFmpeg reads an unguessable, temporary loopback HTTP endpoint serving only that opened file; proxy use, file protocols, playlists, and arbitrary input demuxers are excluded. Processes run without a shell and with hidden windows.

`getTrickplay` returns `null` for absent or invalid cached work. `requestTrickplay` starts work and returns `pending`, or returns existing `ready`/`failed` work. Concurrent requests for the same source share one job. Failed work has a readable error and a one-minute retry delay; `{ retry: true }` explicitly retries it. Cancelled work is immediately retryable after a restart. Tool stderr and source paths are not exposed in preview errors.

`getSprite(item, root, index)` accepts an integer sheet index and returns `{ path, size, mimeType, cacheKey }`, or `null`. Only the server should use this filesystem path. Add authenticated URLs to the manifest's `images` array, and include `cacheKey` in image URLs or cache validators so a changed file cannot reuse a browser's old image. Images require the same authorization as video playback.

## Sprite layout

A ready manifest looks like:

```json
{
  "version": 1,
  "status": "ready",
  "cacheKey": "<64-character file signature>",
  "duration": 1440,
  "interval": 5,
  "width": 160,
  "height": 90,
  "columns": 10,
  "rows": 10,
  "thumbnailCount": 288,
  "images": [
    { "index": 0, "start": 0, "end": 500, "thumbnailCount": 100, "bytes": 123456 },
    { "index": 1, "start": 500, "end": 1000, "thumbnailCount": 100, "bytes": 123456 },
    { "index": 2, "start": 1000, "end": 1440, "thumbnailCount": 88, "bytes": 112233 }
  ]
}
```

For hover time `t`, use `thumbnail = clamp(floor(t / interval), 0, thumbnailCount - 1)`. The sheet index is `floor(thumbnail / 100)`, column is `thumbnail % 10`, and row is `floor((thumbnail % 100) / 10)`. Display a 160 × 90 crop with background position `-column * 160`, `-row * 90`. Each sheet is 1600 × 900; unused final tiles must never be selected.

One sequential FFmpeg pass samples the video, scales it with aspect ratio preserved, pads it, and tiles it into JPEG sheets. The interval is `max(5, ceil(duration / 720))`: ordinary episodes use five seconds, a two-hour movie uses ten, and longer media use a larger interval to retain the hard 720-thumbnail limit. FFmpeg's reported sampled-frame count determines the manifest, including media whose video stream ends before its audio. Audio-only media fail gracefully.

## Intro evidence and playback

There are two separate sources:

* **Named chapters:** `detectIntroFromChapters(chapters, duration)` returns the file's explicit opening boundaries with `source: 'chapters'`, `confidence: 1`, and `confirmed: true`. It accepts raw FFprobe chapters (`start_time`, `end_time`, `tags.title`) or normalized chapters (`start`, `end`, `title`). Missing chapter ends can be inferred from the next chapter. Recognized names include Intro, Introduction, Opening, Opening credits/theme/song, OP/OP1, オープニング, イントロ, 片头, and 片頭. Valid spans are 2–300 seconds, at most 80% of the media, and begin within the first ten minutes and first half of the file. Unnamed chapter numbers, recaps, prologues, and cold opens are not opening evidence.
* **Repeated audio:** `detectSharedIntros(items, { signal, searchSeconds: 600 })` compares episodes supplied by the server. Group items by series and season before calling it. It returns `{ segments, analyzed, failures, skipped }`; `segments` is keyed by episode ID, and unmatched episodes have no segment. Returned fingerprints always have `source: 'fingerprint'` and **`confirmed: false`**. Offer these as suggestions that the user can skip manually or review. Automatic skipping must require a trusted chapter, manual boundaries, or the user's explicit confirmation of a suggested segment.

An example suggestion is:

```json
{
  "start": 82.125,
  "end": 171.75,
  "source": "fingerprint",
  "confidence": 0.986,
  "confirmed": false,
  "evidence": {
    "algorithm": "hoshi-spectral-v1",
    "matchedEpisodes": ["episode-2"],
    "similarity": 0.993,
    "coverage": 0.993,
    "matches": ["<per-peer boundary and quality records>"]
  }
}
```

Audio is decoded to mono 8 kHz PCM. A portable spectral matcher compares normalized frequency profiles from overlapping 512 ms windows sampled every 125 ms. Multiple locality-sensitive hashes propose matching offsets; full spectral comparisons then verify continuous shared sections. Matches must last 20–180 seconds and no more than 45% of either episode, have at least 90% matching coverage and 0.975 average similarity among matching frames, and contain changing audio. Silence, static tones, short logos, unrelated episodes, and whole-file duplicates are rejected in the deterministic tests. When several peers agree, the returned segment uses the intersection of their boundaries.

This is Hoshi's own spectral matcher, not Chromaprint. It works with ordinary FFmpeg builds without a native fingerprint library. The numerical confidence is a match-quality score, not a calibrated probability that a section is an opening. Repeated recaps, advertisements, or music used in the story can still resemble an opening; alternate audio, overlapping dialogue, heavy edits, or tempo changes may produce no match. It does not guess a fixed opening duration, and it cannot discover repeated audio from only one episode. Boundary precision is approximate; user adjustments take precedence.

## Resource limits and persistence

* The default queue runs one process/job at a time and accepts twelve waiting jobs. `concurrency` can be set to 1 or 2, and `maxQueuedJobs` is capped at 32. Queue overflow returns `ANALYSIS_BUSY` (429).
* Only one opening analysis may run per service. It processes at most twelve distinct episode IDs/files; additional items are reported in `skipped`. The caller can run bounded groups sequentially. Fingerprints process one episode at a time and at most the first ten minutes; `searchSeconds` can reduce this to 30–600 seconds.
* Audio output is capped at approximately 9.6 MB per decoded episode. Compact fingerprints persist under `fingerprints/`; PCM is never written to disk. Decoding is capped at three minutes per episode. A preview's default timeout is twenty minutes; `processTimeoutMs` configures it, bounded to one hour.
* Cache keys include the canonical media path, modification time, size, and algorithm version. Replacing or editing media generates a different key. Files are revalidated before processing and afterwards. Cached fingerprints are reused across analyses and server restarts.
* The default completed cache budget is 256 MiB, configurable through `maxCacheBytes` from 1 MiB to 1 GiB. On writes, oldest completed entries are evicted to honor the budget. Each sprite entry is limited to 16 MiB or the smaller configured budget. An active job's temporary output can briefly exist in addition to the completed cache; a size monitor and hard thumbnail count bound it.
* A sprite manifest becomes ready only after every output sheet is complete. Partial directories are removed on failure/shutdown. Abandoned partials older than a day are removed at startup. Cache directories and served files cannot be symbolic links. Unrecognized cache contents are never recursively deleted.
* `shutdown()` cancels queued work, kills active FFmpeg/FFprobe children, waits for cleanup, and aborts opening comparisons. Pass an `AbortSignal` to cancel an individual opening analysis.

Opening suggestions and user confirmation are application records, not stored in the fingerprint cache. Preserve manual overrides and confirmed choices when background analysis finishes. Let the server's library job status report partial failures instead of marking an entire group successful when a tool or episode failed.

## Verification

Run `node --test test/intelligence.test.mjs` for deterministic synthetic-audio tests and process/cache lifecycle checks. To include actual sprite and AAC fingerprint integration:

```powershell
$env:HOSHI_FFMPEG_TEST_PATH = 'C:\path\to\ffmpeg.exe'
node --test test/intelligence.test.mjs
```

The integration test generates its own media, decodes the resulting JPEG, verifies two matching openings at different timestamps after AAC encoding, and checks persistent fingerprint reuse. It uses no copyrighted media and does not require FFprobe because it supplies generated media durations.

## References

FFmpeg documents the [fps filter](https://ffmpeg.org/ffmpeg-filters.html#fps), [scale filter](https://ffmpeg.org/ffmpeg-filters.html#scale), and [tile filter](https://ffmpeg.org/ffmpeg-filters.html#tile) used to generate the sprite sheets. Its [Chromaprint muxer documentation](https://ffmpeg.org/ffmpeg-formats.html#chromaprint) describes a separate fingerprint facility available in appropriately built FFmpeg binaries. The [Intro Skipper project's detection overview](https://github.com/intro-skipper/intro-skipper/wiki/#detection-types) explains the established distinction between encoder-authored chapter markers and recurring audio evidence; Hoshi's implementation follows that distinction while using its own portable matcher.
