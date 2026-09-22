# Third-party components

Hoshi is an independent implementation. It does not include Jellyfin source code, assets, branding, or its license. References to Jellyfin describe the product category and research context.

Standalone Hoshi executables include Node.js, which contains SQLite and other third-party components. Every packaged archive includes `NODE-LICENSE.txt` with the license and notices from the exact Node.js version used for that executable. Retain that file when redistributing an archive.

Build-time tools are esbuild (MIT) and postject (MIT). They are used to produce the executable and are not shipped as separate runtime packages. Their upstream sources and licenses are available at [esbuild](https://github.com/evanw/esbuild) and [postject](https://github.com/nodejs/postject).

FFmpeg and FFprobe are external executables and are not included in Hoshi's standalone archives. Their licensing depends on the particular build. The default local Docker build installs the distribution's FFmpeg package and DejaVu fallback fonts; operators can disable that option. Those package licenses remain in the locally built image. See [FFmpeg project information](https://ffmpeg.org/).

No open-source license has been assigned to Hoshi's own application source by these third-party notices. Keeping the source repository private and distributing binaries are separate choices from licensing the application.

## Metadata providers

Hoshi includes an unmodified approved TMDB logo from https://www.themoviedb.org/assets/v4/logos/v2/blue_short-8e7b30f73a4020692ccca9c88bafe5dcb6f8a62a4c6bc55cd9ba82bb2cd95f6c.svg for API attribution, accessed 2026-09-22. TMDB owns its mark. Provider data is fetched separately by the user’s server and is not bundled with binaries. TVmaze metadata is CC BY-SA; Kitsu and TMDB retain their respective terms. See PROVIDERS.md and Settings → Metadata & credits.
