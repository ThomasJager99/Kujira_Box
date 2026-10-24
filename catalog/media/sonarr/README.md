# Sonarr

**Sonarr** is a TV series collection manager — the "*arr" app that automates
following shows. You add a series once and Sonarr monitors for new episodes,
grabs them automatically through your indexers (via **Prowlarr**) and download
client, then renames and sorts the files into a tidy library your media server
(Jellyfin/Plex/Emby) can read. It handles seasons, episode numbering, quality
profiles (upgrade SD → HD automatically), release filtering, and tells you about
calendar airings and missing episodes. For a homelab running Jellyfin + Prowlarr,
Sonarr is the piece that keeps your TV library current without manual work.


<p align="center">
  <img src="../../../assets/logos/sonarr.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `lscr.io/linuxserver/sonarr` (LinuxServer.io)
- **Project:** https://github.com/Sonarr/Sonarr
- **Docs:** https://wiki.servarr.com/sonarr

## Deployment Notes

- **One container.** Single `sonarr` service (LinuxServer image, Alpine-based).
- **File ownership.** Runs under `PUID`/`PGID` (default `1000:1000`) — set these
  to match the user that owns your media and downloads folders.
- **Single HTTP interface** on port `8989`.
- **Persistent state.** Named volume `sonarr_config` (`/config`) holds the app
  database and settings. Your **media** (`/tv`) and **downloads** (`/downloads`)
  are bind-mounted from host paths you choose.
- **Hardlinks / atomic moves (important).** Put `/tv` and `/downloads` under one
  shared parent folder on the **same filesystem** and mount that single parent,
  so Sonarr can **hardlink or instant-move** finished downloads into the library
  instead of making slow full copies (and it won't double your disk usage). If
  they're on different mounts, every import is a copy.
- **Connecting the stack.** Add your download client and connect Prowlarr under
  *Settings* so indexers sync in; Sonarr and the other apps must be able to reach
  each other on the network.
- **Pinned version.** LinuxServer publishes `latest` plus release tags
  (`4.x.x-lsNNN`); pin one for reproducible rebuilds.
- **Healthcheck** fetches `/ping` with the image's busybox `wget`.
- **Optional hardening** (commented): `cap_drop: [ALL]`, `read_only: true`,
  `tmpfs: [/tmp]` — enable after testing (LSIO images often need a writable
  root filesystem).
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Follow TV shows and auto-download new episodes as they air.
- Automatically upgrade episodes to a better quality when available.
- Rename and organise files into a clean, media-server-friendly structure.
- See a calendar of upcoming airings and a list of missing episodes.

## Thanks

To the **Sonarr** team and **LinuxServer.io** for the series manager and the
well-maintained image.

## Links

- Project: https://github.com/Sonarr/Sonarr
- Wiki: https://wiki.servarr.com/sonarr
- Image: https://docs.linuxserver.io/images/docker-sonarr
