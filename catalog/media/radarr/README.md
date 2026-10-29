# Radarr

**Radarr** is a movie collection manager — the "*arr" app for film, a fork of
Sonarr built for movies instead of TV. You add movies to your library (or whole
lists), and Radarr monitors for a release in the quality you want, grabs it
automatically through your indexers (via **Prowlarr**) and download client, then
renames and sorts the file into a clean library your media server
(Jellyfin/Plex/Emby) can read. It handles quality profiles and upgrades, custom
formats, lists (e.g. "all of a director's films"), and tells you what's missing.
For a homelab running Jellyfin + Prowlarr, Radarr is the piece that keeps your
movie library current without manual work.



<p align="center">
  <img src="../../../assets/logos/radarr-4k.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `lscr.io/linuxserver/radarr` (LinuxServer.io)
- **Project:** https://github.com/Radarr/Radarr
- **Docs:** https://wiki.servarr.com/radarr

## Deployment Notes

- **One container.** Single `radarr` service (LinuxServer image, Alpine-based).
- **File ownership.** Runs under `PUID`/`PGID` (default `1000:1000`) — set these
  to match the user that owns your media and downloads folders.
- **Single HTTP interface** on port `7878`.
- **Persistent state.** Named volume `radarr_config` (`/config`) holds the app
  database and settings. Your **movies** (`/movies`) and **downloads**
  (`/downloads`) are bind-mounted from host paths you choose.
- **Hardlinks / atomic moves (important).** Put `/movies` and `/downloads` under
  one shared parent folder on the **same filesystem** and mount that single
  parent, so Radarr can **hardlink or instant-move** finished downloads into the
  library instead of making slow full copies (and it won't double your disk
  usage). If they're on different mounts, every import is a copy.
- **Connecting the stack.** Add your download client and connect Prowlarr under
  *Settings* so indexers sync in; Radarr and the other apps must be able to reach
  each other on the network.
- **Pinned version.** LinuxServer publishes `latest` plus release tags
  (`5.x.x-lsNNN`); pin one for reproducible rebuilds.
- **Healthcheck** fetches `/ping` with the image's busybox `wget`.
- **Optional hardening** (commented): `cap_drop: [ALL]`, `read_only: true`,
  `tmpfs: [/tmp]` — enable after testing (LSIO images often need a writable
  root filesystem).
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Add movies and auto-download them in your chosen quality when available.
- Automatically upgrade a film to a better release later (quality profiles).
- Import lists (a franchise, a director, IMDb/Trakt lists) and fill them in.
- Rename and organise files into a clean, media-server-friendly structure.

## Thanks

To the **Radarr** team and **LinuxServer.io** for the movie manager and the
well-maintained image.

## Links

- Project: https://github.com/Radarr/Radarr
- Wiki: https://wiki.servarr.com/radarr
- Image: https://docs.linuxserver.io/images/docker-radarr
