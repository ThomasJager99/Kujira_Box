This directory contains an example deployment of Jellyfin, the free software media
server — a fully open-source alternative to Plex or Emby with no premium tiers,
paywalls or telemetry. Jellyfin organizes your own movies, TV shows, music and
photos into a polished library and streams them to almost any device: a web
browser, phone and tablet apps, smart TVs, Android TV, Roku, Kodi and more. It
fetches artwork and metadata automatically, supports multiple users, on-the-fly
transcoding, and even live TV and DVR with a tuner — all running on hardware you
control, with your library never leaving your server.


<p align="center">
  <img src="../../../assets/logos/jellyfin.svg" width="180">
</p>

<br>


## Official container image

https://github.com/jellyfin/jellyfin — image published as `jellyfin/jellyfin`.

## Deployment Notes

- Runs as a single container — server, web UI and transcoder in one.
- Stores its configuration and metadata database in the `jellyfin_config` volume
  and transient transcoding data in `jellyfin_cache`.
- Your media library is a **read-only bind mount** (`MEDIA_PATH` → `/media:ro`) —
  Jellyfin only reads your files and never modifies them.
- Exposes the web interface on port `8096`, plus `7359/udp` for LAN client
  auto-discovery (and optionally `1900/udp` for DLNA); intended to sit behind a
  reverse proxy for TLS when reached over the internet.
- `JELLYFIN_PublishedServerUrl` is set so the links and stream URLs Jellyfin
  hands to clients point at the right address.
- Includes a healthcheck against Jellyfin's `/health` endpoint.
- Memory and CPU limits are set (transcoding is CPU-heavy — give it cores if you
  have them). **Hardware transcoding** (Intel QSV / VAAPI) is available by passing
  `/dev/dri` — see the commented block in the compose file.
- Runs as root by default; a non-root `user:` line is provided (commented) — enable
  it after giving that user ownership of the config/cache dirs.
- **Backups:** the `jellyfin_config` volume holds all settings, users and metadata;
  back it up. The `jellyfin_cache` is disposable, and your media is already your own.

## Example Use

Jellyfin can be used to:

- stream your movies, TV shows and music to any device, in or out of the house
- use native apps on phones, tablets, Android TV, Roku, Kodi and the web
- give family members their own accounts, watch state and parental controls
- transcode on the fly so anything plays on anything (with optional GPU accel)
- watch and record live TV with a tuner (DVR)
- replace a paid Plex/Netflix-style setup with a fully self-hosted, telemetry-free one

This gives a household its own private streaming service, built entirely from
open-source software.

## Thanks

Thanks to the Jellyfin project and its community — a volunteer-driven fork that
turned into a genuinely excellent, completely free media server with no strings
attached.

## Links

- Source: https://github.com/jellyfin/jellyfin
- Website: https://jellyfin.org
- Documentation: https://jellyfin.org/docs
