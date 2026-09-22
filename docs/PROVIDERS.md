# Metadata providers

Hoshi has three built-in, read-only adapters. Anime and television lookups work without an account or API key. Local media playback does not depend on a metadata service.

| Provider | Default use | Credentials | Minimum spacing |
| --- | --- | --- | --- |
| Kitsu | Anime titles, descriptions, categories, posters, cover art, episodes and thumbnails | None | 1,000 ms |
| TVmaze | Television titles, descriptions, genres, aliases, posters, backgrounds and episodes | None | 550 ms |
| TMDB | Optional movie and television metadata | Server-side API Read Access Token | 250 ms |

The adapters only request official JSON API endpoints. They do not scrape websites, upload media, synchronize watchlists, load remote JavaScript, or download a provider's entire catalog. Searches disclose a title query and optional release year; detail requests disclose provider IDs and, when applicable, episode numbers. Requests also expose the server's IP address and Hoshi user agent to the provider.

## Configuration

Kitsu and TVmaze are available immediately. To enable TMDB, set `HOSHI_TMDB_TOKEN` in the server environment to your **API Read Access Token**, then restart the server. This is the long token from TMDB's account API settings. Hoshi sends it in an `Authorization: Bearer …` header; it is never inserted into a URL, returned by the provider catalog, or included in error messages. Do not put the token in an `.nfo` file, a media folder, frontend code, a URL, or a repository.

TMDB remains disabled when this variable is empty. The other two providers continue working. The metadata settings switch controls online lookups in the application; downloaded metadata and artwork can remain useful offline.

## Attribution and usage notes

Provider records retain `provider` and `sourceUrl`. Display the source link with the title's details and keep provider credits in Settings/About. Text is converted to plain text, field sizes are bounded, and ratings are normalized to a ten-point scale. These are transformations of the returned metadata; they do not create a new content license.

### TVmaze

The [official public API documentation](https://www.tvmaze.com/api) licenses API use under CC BY-SA and asks applications to credit TVmaze, for example by linking to the relevant show or episode page. Preserve that attribution and applicable ShareAlike obligations when redistributing adapted metadata. The same page documents a minimum allowance of 20 requests per ten seconds per IP, instructs clients to back off on HTTP 429, and recommends image caching. Hoshi spaces requests by at least 550 ms, serializes access, and caches responses.

Suggested credit: **TV metadata from [TVmaze](https://www.tvmaze.com), licensed under [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0/).** Keep the record's own source link as well. See TVmaze's [copyright policy](https://www.tvmaze.com/site/copyright) for content provenance and removal information. Metadata and artwork availability can change.

### Kitsu

Kitsu's [official JSON:API documentation](https://hummingbird-me.github.io/api-docs/) describes public `GET` endpoints without authentication, text search, related categories, and pagination. Its [official API repository](https://github.com/hummingbird-me/api-docs) identifies those docs and the older Apiary documentation. Production requests use `https://kitsu.app/api/edge`, verified live on September 22, 2026; some documentation still displays the former `kitsu.io` domain. Unauthenticated responses omit content that Kitsu requires a mature-content-enabled account to access.

Hoshi uses bounded personal lookups and credits **Anime metadata and artwork from [Kitsu](https://kitsu.app)**, with a link to the particular anime. It does not infer an unrestricted content license from the Apache 2.0 license attached to Kitsu's software or documentation.

The [public terms page](https://kitsu.app/terms) requires JavaScript. The reviewed [terms source in Kitsu's own web repository](https://github.com/hummingbird-me/kitsu-web/blob/the-future/markdown/terms.md) is labeled Version 1.0, December 1, 2016. It grants personal, revocable access, restricts separate commercial use and redistribution of Kitsu content, distinguishes user content, and mentions separate API terms available on request. That source is not evidence of a blanket commercial or redistribution license. Review the provider's current terms before using metadata outside a personal server or distributing a data collection. These are implementation notes, not a legal opinion.

### TMDB

The [official TMDB FAQ](https://developer.themoviedb.org/docs/faq) describes noncommercial API use with attribution and directs commercial users to TMDB for licensing. When enabled, the application must display an [approved TMDB logo](https://www.themoviedb.org/about/logos-attribution) and this notice in its About/Credits area:

> This product uses the TMDB API but is not endorsed or certified by TMDB.

Keep TMDB branding less prominent than Hoshi branding. A plain text source link alone does not satisfy the logo requirement. See [TMDB's API terms](https://www.themoviedb.org/api-terms-of-use) and the [authentication documentation](https://developer.themoviedb.org/docs/authentication-application). Supplying a token does not itself establish permission for every use of images or data.

### Why AniList is excluded

AniList's [official API terms](https://docs.anilist.co/guide/terms-of-use) explicitly prohibit competing, non-complementary anime or manga list/tracker services, covering media data as well as user data. Hoshi includes a personal anime watchlist, so it does not silently use AniList as a metadata backend. No AniList credentials or requests are implemented.

## Adapter contract

`src/providers.mjs` exports `createMetadataProviders`, `isAllowedArtworkUrl`, `plainMetadataText`, and `MetadataProviderError`. There are no runtime npm dependencies.

```js
const providers = createMetadataProviders({
  fetchImpl: globalThis.fetch,                // Optional: inject a standards-compatible fetch for tests.
  tmdbToken: process.env.HOSHI_TMDB_TOKEN,    // Optional; empty means disabled.
  signal: serverShutdownSignal,              // Optional lifetime cancellation signal.
});

const results = await providers.search({
  query: 'Cowboy Bebop', type: 'anime', year: 1998,
  // provider: 'kitsu', signal: requestSignal,
});
const anime = await providers.get({provider: 'kitsu', id: results[0].id});
const episode = await providers.get({provider: 'kitsu', id: anime.id, season: 1, episode: 1});
const episodes = await providers.getEpisodes({provider: 'kitsu', id: anime.id, season: 1});
const catalog = providers.available();
await providers.shutdown();
```

`search` accepts `type: 'anime' | 'series' | 'movies'`. Defaults are Kitsu, TVmaze, and TMDB respectively. An explicit provider must support the requested type. TVmaze can also be selected explicitly for anime. TMDB accepts movie or television types, so an anime library must establish the correct movie/television identity before using a TMDB ID.

Searches return at most 20 normalized records. Kitsu and TVmaze results are filtered locally by a supplied year, retaining entries whose release year is unknown; TMDB additionally receives its documented year parameter. An empty result is a successful search with no matches. Matching confidence and any automatic assignment belong to the metadata service, not these adapters.

`get` returns a title record unless **both** `season` and `episode` are supplied, in which case it returns an episode. IDs are positive integers up to 2,147,483,647, returned as strings. Title lookup defaults are `anime` for Kitsu, `series` for TVmaze, and `movies` for TMDB. Always pass `type: 'series'` for a TMDB television title or episode.

Title records contain `provider`, `id`, `type`, `title`, `originalTitle`, `aliases`, `year`, `synopsis`, `genres`, `rating`, `runtime`, `posterUrl`, `backdropUrl`, `sourceUrl`, and `status`. Some providers additionally supply `episodeCount`, `seasonCount`, `format`, or `language`. Ratings are 0–10; runtimes are minutes. Missing numerical/artwork fields are `null`, missing prose is an empty string, and missing collections are empty arrays. Search records may lack fields available from a detail request.

Episode records contain `provider`, `id`, `showId`, `season`, `episode`, `title`, `synopsis`, `airDate`, `imageUrl`, `runtime`, and `sourceUrl`. Season zero is supported. Unnumbered specials are omitted because they cannot be mapped reliably to a local episode number. Provider season/episode numbering must not be assumed interchangeable with absolute anime numbering or an alternative DVD order.

`getEpisodes` returns sorted, deduplicated episodes and can filter a season. TVmaze's list is capped at 5,000 records and 4 MiB. Kitsu requests pages of 20, up to 100 pages (2,000 episodes); it constructs every page URL itself and never follows a returned URL. A title exceeding that limit fails explicitly instead of silently returning a partial list. TMDB can retrieve one season or enumerate up to 100 seasons and 5,000 episodes. Each HTTP request is bounded separately; a large uncached episode list can take longer than one request timeout. Callers should use a cancellation signal and cache the completed normalized list.

`available()` returns descriptors with `id`, `name`, `types`, `enabled`, `requiresKey`, `attribution`, `website`, and applicable terms/license/logo-information links. It never includes credentials. `shutdown()` cancels work, clears the in-memory cache, and rejects subsequent reads.

## Request and artwork boundaries

- JSON requests use fixed HTTPS origins and internal endpoint templates. Redirects are rejected. Query strings are encoded using `URLSearchParams`; response links never become fetch destinations.
- Every read has a ten-second deadline covering queueing, retries, response headers, and streamed JSON. The default body limit is 2 MiB, increased to 4 MiB only for TVmaze episode lists. Incorrect MIME types, invalid UTF-8/JSON, and incompatible response schemas fail safely.
- HTTP 429 and 503 have at most one retry, after provider spacing and `Retry-After`. A wait exceeding three seconds or the remaining deadline is reported to the caller without retrying early. Provider response bodies and raw network errors are not exposed in errors.
- Identical reads share one in-flight transport. Individual consumers can cancel independently; the transport is aborted when its last consumer leaves. Up to 64 distinct reads can be queued. Provider queues are independent and serialize each provider's requests.
- The LRU/TTL memory cache defaults to 256 entries, 16 MiB of received JSON, and one hour. Errors are not cached. Returned normalized records cannot mutate cached raw responses. Application-level persistent caching is separate.
- `timeoutMs`, `cacheTtlMs`, `maxCacheEntries`, `maxCacheBytes`, and `minIntervals` may be injected for tests or internal configuration. Request intervals cannot be lowered below the table above, and timeout cannot exceed ten seconds.
- All titles, summaries, aliases, and genres are plain text. Scripts, tags, encoded tags, comments, and control bytes are removed. Render them with normal text escaping, never `innerHTML`.
- `isAllowedArtworkUrl(provider, url)` accepts only raster image paths on exact HTTPS CDN hosts: `static.tvmaze.com`/`e.gstatic.tvmaze.com`, `media.kitsu.app`/`media.kitsu.io`, and `image.tmdb.org`. Credentials, unusual ports, fragments, SVGs, and arbitrary hosts are rejected. The helper validates a URL; it does not fetch it or inspect image bytes. The artwork download service must separately enforce redirects, response size, deadlines, image signatures, and safe filesystem writes.

## Verification and API references

`node --test test/providers.test.mjs` covers realistic provider schemas, thumbnails, sanitization, keys, fixed hosts, response limits, rate spacing, retry caps, caching, cancellation, shutdown, timeouts, and safe errors.

Live no-key smoke checks on September 22, 2026 successfully retrieved Cowboy Bebop from Kitsu with description, categories, poster, cover art, and the Asteroid Blues episode thumbnail; they also retrieved Severance from TVmaze with description, aliases, poster, background, and the Good News About Hell episode image. TMDB is tested with realistic mocked API responses; no personal token was required or used for the smoke check.

Primary endpoint references: [TVmaze API](https://www.tvmaze.com/api), [Kitsu JSON:API](https://hummingbird-me.github.io/api-docs/), [Kitsu episode filter implementation](https://github.com/hummingbird-me/kitsu-server/blob/master/app/resources/episode_resource.rb), [TMDB movie search](https://developer.themoviedb.org/reference/search-movie), [TMDB TV episode details](https://developer.themoviedb.org/reference/tv-episode-details), and [TMDB image URLs](https://developer.themoviedb.org/docs/image-basics).
