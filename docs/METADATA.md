# Automatic descriptions and artwork

Hoshi 0.3 adds built-in metadata providers. After adding a library and scanning it, title lookup runs in the background. Existing libraries are checked when the upgraded server starts. A periodic job checks every six hours for records due for refresh. It does not rescan your filesystem.

| Library | Default provider | Configuration |
| --- | --- | --- |
| Anime | Kitsu | No account or API key |
| Series | TVmaze | No account or API key |
| Movies | TMDB | Your own TMDB API Read Access Token |
| Any supported video library | Local NFO files | No internet required |

Provider results include descriptions, posters, backdrops, genres, ratings, series status, episode counts, and episode titles/synopses/air dates/images when available. Browser artwork requests stay on your Hoshi server; provider images are downloaded into a bounded local cache. An episode without a matching provider episode retains its filename title and can use the series description. Music tag providers are not included.

## Organize your folders

Recommended examples:

```text
Anime/
  Cowboy Bebop (1998)/
    Season 01/
      Cowboy Bebop S01E01.mkv
      Cowboy Bebop S01E02.mkv
Series/
  Severance (2022) [tvmazeid-44933]/
    Season 01/
      Severance S01E01.mkv
Movies/
  Arrival (2016) [tmdbid-329865]/
    Arrival (2016).mkv
```

Titles are inferred from folders and filenames, with common release tags and episode markers removed. Years distinguish remakes; sequel numbers are retained. Supported explicit folder IDs include `tmdbid`, `tvmazeid`, and `kitsuid`. IMDb IDs can be read from NFO files but are not resolved by these adapters. Kitsu models many anime seasons as separate titles; Hoshi will not automatically use season one as a confident match for a later-season folder. Use **Identify title** for uncertain seasons, alternate numbering, specials, or anime movies. TMDB supports movie and series libraries; anime libraries use Kitsu or TVmaze. Combined multi-episode files do not yet have separate playable virtual episodes.

## Review and correct matches

Open a title and find **Story & sources**. **Identify title** searches supported providers, shows candidates and years, and lets you select the correct result. A manual selection locks the identity. Apply it to the group to identify related episodes at once; anime groups are season-specific. Same-title shows with different years remain separate. **Unlock identity** allows automatic matching again. **Refresh metadata** updates provider fields while keeping the chosen locked identity.

**Settings → Metadata & credits → Review matches** lists up to 100 pending, ambiguous, unmatched, or failed titles. The **Metadata** button beside a library shows background progress and starts a refresh. Jobs and provider failures do not block playback. A provider outage retains the last good result. Administrator access is required to change shared metadata; viewers can read descriptions and source credits.

Priority is **manual field edits → local NFO → provider data → filename**. A deliberately blank manual synopsis or release year remains blank. **Reset manual edits** restores local/provider values. Identity locking and field editing are separate: a lock fixes which title is selected; fields can still refresh from that title. Metadata refreshes do not change watchlists, playback progress, video files, subtitles, or audio tracks.

## Local files

Put `tvshow.nfo` in the show's folder. Episode metadata can use the video's basename plus `.nfo`; movie metadata can use the basename plus `.nfo` or `movie.nfo`. Hoshi reads common title, plot, year, genre, rating, show, season, episode, premiered/aired, and typed unique-ID fields. A show NFO's title becomes the series title, not every episode title. Show descriptions and episode descriptions remain distinct. Episode provider IDs cannot silently replace the show's ID.

Local artwork can use `poster.jpg`, `folder.jpg`, `cover.jpg`, a media basename image, or PNG/WebP equivalents. Parent folders are checked up to the library root. Backdrops use `fanart`, `backdrop`, or `background`; episode thumbnails can use a basename image or `basename-thumb`. Local artwork takes priority over provider artwork, followed by a generated frame when FFmpeg is available. NFO URLs are never fetched, and unsupported XML entities, DTDs, external links, oversized files, and symlinks are rejected.

## Optional TMDB setup

Obtain an API Read Access Token from your own [TMDB account settings](https://www.themoviedb.org/settings/api), review its terms, then set `HOSHI_TMDB_TOKEN` in the environment of the server process and restart Hoshi. This is the Bearer/read-access token, not an API v3 key. For systemd use a protected environment file; for Docker pass it through your deployment's environment or secret management. Do not put it in a repository or library folder. It is used only by the server and never returned through the settings API or embedded in public builds.

## Privacy, caching, and limits

Online lookup sends normalized title searches and provider IDs, not paths or video content, to the selected provider. Settings can disable online metadata and automatic refresh separately. `HOSHI_METADATA_ENABLED=0` enforces offline mode even if an administrator's saved setting enables lookup. NFO files and already cached artwork remain usable offline. Turning online lookup off aborts active provider requests; an already started artwork download may finish. No anime tracking account is linked or synchronized.

Successful title/episode data is cached on disk for seven days, searches and empty results for one day, with up to 2,000 records. Explicit refresh bypasses the persistent metadata cache; the adapter may reuse an identical request for up to one hour to respect providers' limits. Artwork has a 256 MiB cache budget and an 8 MiB per-download limit. Downloads permit only fixed provider HTTPS CDNs, reject redirects and non-raster responses, and have time/size/concurrency limits. Provider requests have bounded retry, spacing, timeout, and cancellation. Pending jobs are marked interrupted after a restart; automatic jobs can resume from cached records.

These are built-in adapters, not support for installing Jellyfin's plugin binaries. Broad multilingual metadata preference, season artwork browsing, arbitrary external plugins, metadata export, and full Jellyfin parity remain future work. See [provider research and attribution](PROVIDERS.md).

## Validation for 0.3

Automated tests cover normalization, ambiguous remakes, local NFO precedence and hostile XML, admin-only mutation, explicit empty overrides, persistence, failed-provider retention, shared caches, concurrent manual/automatic selection, and safe artwork fetching. Live Kitsu and TVmaze requests verified descriptions and episode artwork. TMDB uses mocked API responses in automated testing because Hoshi ships without an owner's token. Chrome browser checks exercise identification, lock/reset, library refresh, actual cached provider images, and desktop/mobile layouts. Each release platform runs the full test suite and executable startup checks; Linux also runs real FFmpeg media tests and the x64 container smoke test.
