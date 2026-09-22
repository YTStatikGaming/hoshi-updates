# Install and operate Hoshi

Download the matching archive and checksum from [the public releases page](https://github.com/YTStatikGaming/hoshi-updates/releases). The examples use version `0.3.0`; substitute the version you downloaded. Run Hoshi as a normal user with read access to media and write access to its data directory.

## Windows x64

1. Compare the archive's SHA-256 with the text in its `.sha256` file:

   ```powershell
   Get-FileHash .\hoshi-0.3.0-windows-x64.zip -Algorithm SHA256
   Get-Content .\hoshi-0.3.0-windows-x64.zip.sha256
   ```

2. Extract the archive to a folder you own. In PowerShell, change to the extracted folder and start it:

   ```powershell
   .\hoshi.exe
   ```

3. Open `http://localhost:8096`, enter the setup token printed in the terminal, and create an administrator. Usernames contain 3–40 letters, digits, dots, dashes or underscores; passwords contain 12–256 characters.
4. Add a library using a server path such as `D:\Media\Anime`. The current Windows account needs read access. Use an absolute UNC path for a reachable share if appropriate; scheduled tasks often cannot see interactive mapped drive letters.

To change settings for the current PowerShell session:

```powershell
$env:HOST = '0.0.0.0'
$env:PORT = '8096'
$env:HOSHI_DATA_DIR = 'D:\HoshiData'
.\hoshi.exe
```

The default data folder is `%LOCALAPPDATA%\Hoshi\data`. Keep the terminal open while the server is running. For persistent startup, create a Windows Task Scheduler task using your dedicated server account, with `hoshi.exe` as the program and its folder as the working directory. Store persistent variables on that account or use a script with explicit paths. The executable is a console program, not a native Windows service; do not register it with `sc.exe` as if it implemented the Service Control Manager protocol.

If Windows reports an unsigned application, verify its source and checksum before deciding to run it. No system-wide SmartScreen change is needed.

## Ubuntu Server and other compatible Linux

Use the x64 archive on Intel/AMD machines and the ARM64 archive on 64-bit ARM machines. Release x64 builds use Ubuntu 22.04 and ARM64 builds use Ubuntu 24.04; older distributions are not certified. Alpine/musl and 32-bit Linux binaries are not provided.

```sh
sha256sum -c hoshi-0.3.0-linux-x64.tar.gz.sha256
tar -xzf hoshi-0.3.0-linux-x64.tar.gz
cd hoshi-0.3.0-linux-x64
./hoshi
```

Complete setup at `http://localhost:8096`. For a headless server, an SSH tunnel keeps initial setup private:

```sh
ssh -L 8096:127.0.0.1:8096 your-user@your-server
```

Then open `http://localhost:8096` on your own computer. To listen on the LAN instead:

```sh
HOST=0.0.0.0 PORT=8096 HOSHI_DATA_DIR=/path/you/own/hoshi-data ./hoshi
```

### Optional systemd service

From the extracted archive directory, install the binary and the supplied service definition:

```sh
sudo useradd --system --user-group --home-dir /var/lib/hoshi --shell /usr/sbin/nologin hoshi
sudo install -d /opt/hoshi /etc/hoshi
sudo install -m 0755 hoshi /opt/hoshi/hoshi
sudo install -m 0644 deploy/hoshi.service /etc/systemd/system/hoshi.service
sudo install -m 0600 deploy/hoshi.env.example /etc/hoshi/hoshi.env
sudo systemctl daemon-reload
sudo systemctl enable --now hoshi
sudo journalctl -u hoshi -n 30 --no-pager
```

If the `hoshi` account already exists, omit the `useradd` command. The startup log contains the first-run setup token. `StateDirectory` creates `/var/lib/hoshi` with the right owner. Edit `/etc/hoshi/hoshi.env` to change settings and use `sudo systemctl restart hoshi` afterward.

The supplied service prevents access to home directories and filesystem writes outside its state directory. Place media under `/srv/media` or another readable mounted folder. Grant the `hoshi` account read and directory-traversal permissions using your existing media group or ACLs. It does not need write access. Keep network mounts available before starting scans; extend the service with mount dependencies if your storage requires them.

Logs: `sudo journalctl -u hoshi -f`. Stop: `sudo systemctl stop hoshi`. Update: stop, back up `/var/lib/hoshi`, replace `/opt/hoshi/hoshi`, and start the service again.

## macOS

Apple Silicon uses the `macos-arm64` archive:

```sh
shasum -a 256 -c hoshi-0.3.0-macos-arm64.tar.gz.sha256
tar -xzf hoshi-0.3.0-macos-arm64.tar.gz
cd hoshi-0.3.0-macos-arm64
./hoshi
```

The data folder defaults to `~/Library/Application Support/Hoshi`. The binary is ad hoc signed, not notarized. If macOS blocks an identified download, follow the per-application approval flow in System Settings → Privacy & Security after checking its origin and checksum. Do not disable Gatekeeper globally.

Intel Macs do not have a standalone release in this alpha. Authorized source users can install Node.js 24 for their architecture and run `npm start` from the source checkout. An Intel distributable installer remains future work.

## Docker Compose on Linux

This route requires access to the private source checkout and Docker with the Compose plugin. It builds locally; no published Hoshi container image is assumed.

Set the media path before starting (the directory must already exist):

```sh
export HOSHI_MEDIA_DIR=/srv/media
docker compose up -d --build
docker compose logs hoshi
```

Open `http://localhost:8096`, use the token in the logs, and add `/media` as the library path. That is the path **inside the container**. The named `hoshi-data` volume stores the database; media is mounted read-only. The container runs as UID/GID 1000, which must be able to read the host media directory.

To publish on the LAN, set `HOSHI_BIND_IP=0.0.0.0` before running Compose. To avoid a port conflict, set `HOSHI_PORT=8097`. Values can be saved in a local `.env` file; keep any setup token private. `docker compose down` preserves the data volume; `docker compose down -v` deletes it.

The locally built image includes distribution FFmpeg, FFprobe and DejaVu fonts for subtitles by default. Set `HOSHI_DOCKER_FFMPEG=false` and rebuild only if you want a minimal image limited to direct playback.

## Playback and FFmpeg

Direct playback depends on the browser's support for the file's container and codecs. H.264 video and AAC audio in MP4 are a practical baseline; file extensions alone do not guarantee playback. Install FFmpeg with the H.264 encoder and AAC encoder available to use the optional software mode. FFprobe supplies richer scan metadata. Official [FFmpeg download guidance](https://ffmpeg.org/download.html) links source, distribution packages, and third-party binary providers.

On Ubuntu/Debian, `sudo apt install ffmpeg fonts-dejavu-core` installs both tools and a fallback font for styled subtitles. On Windows/macOS, install a maintained build from a provider linked by FFmpeg and put it on PATH, or set `HOSHI_FFMPEG_PATH` and `HOSHI_FFPROBE_PATH` to absolute executable paths. Restart Hoshi after changing PATH. Verify with `ffmpeg -version` and `ffprobe -version`. FFmpeg must include libx264, AAC encoding and libass to support every current player feature.

Software transcoding costs CPU. This alpha limits concurrency to two streams and uses progressive fragmented MP4. It supports audio selection, resolution limits and supported subtitle burn-in. Adaptive bitrate, GPU acceleration, HDR tone mapping and surround passthrough remain future work. Seeking starts a new stream at the requested position. Text subtitles may be embedded or stored as matching VTT/SRT/ASS/SSA sidecars. See [the player guide](PLAYBACK.md) for tracks, trickplay and intro skipping.

## HTTPS, backup, and removal

For a reverse proxy, keep Hoshi bound to localhost, proxy HTTP and streaming responses, preserve `Host`, set `HOSHI_ORIGIN=https://your-host`, and set `HOSHI_SECURE_COOKIES=1`. Direct HTTP will not work for login when secure cookies are enabled. Configure proxy timeouts for long playback and avoid buffering full media responses. Do not expose the unencrypted listener to the public Internet.

Back up the whole data directory while Hoshi is stopped; copying only the database file while it is active can miss SQLite write-ahead-log data. Keep that backup and the corresponding older executable together before an upgrade. Database downgrade compatibility is not promised; restore the matching backup if reverting.

To uninstall, stop the process/service and remove the application files. Remove the data folder or Docker named volume only if you also want to delete accounts, settings, catalog, watchlists, and progress. Hoshi never needs to delete your media folders.
