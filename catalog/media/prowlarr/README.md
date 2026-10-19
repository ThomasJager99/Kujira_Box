# Prowlarr

**Prowlarr** is the indexer manager for the "*arr" family (Sonarr, Radarr,
Lidarr, Readarr). Instead of configuring your torrent trackers and Usenet
indexers separately inside every *arr app, you add them **once** in Prowlarr and
it syncs them out to all your connected apps automatically — keeping credentials,
categories and settings consistent in one place. It also gives you a single
search box to query every indexer at once, manage API keys, and track indexer
health. Built on the same codebase and UI as the rest of the *arr stack, it's
the piece that ties your media automation together. For a homelab running
Jellyfin + *arr, Prowlarr is the central indexer hub.


<p align="center">
  <img src="../../../assets/logos/prowlarr.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `lscr.io/linuxserver/prowlarr` (LinuxServer.io)
- **Project:** https://github.com/Prowlarr/Prowlarr
- **Docs:** https://wiki.servarr.com/prowlarr

## Deployment Notes

- **One container.** Single `prowlarr` service (LinuxServer image, Alpine-based).
- **File ownership.** Runs under `PUID`/`PGID` (default `1000:1000`) — set these
  to match the user that owns the config volume/path.
- **Single HTTP interface** on port `9696`.
- **Persistent state.** Named volume `prowlarr_config` (`/config`) holds the app
  database, your indexer definitions and settings. Back it up.
- **Connecting the stack.** In Prowlarr, add your Sonarr/Radarr/etc. under
  *Settings → Apps*; indexers you add then sync out to them. They all need to be
  on a network where they can reach each other (e.g. the same Docker network or
  via host/IP).
- **Pinned version.** LinuxServer publishes `latest` plus release tags
  (`1.x.x-lsNNN`); pin one for reproducible rebuilds.
- **Healthcheck** fetches `/ping` with the image's busybox `wget`.
- **Optional hardening** (commented in compose): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing (LSIO images often
  need a writable root filesystem, so verify before relying on `read_only`).
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Manage all your trackers/indexers in one place and sync them to every *arr app.
- Search every indexer at once from a single interface.
- Keep indexer API keys and categories consistent across the stack.
- Monitor indexer health and failures centrally.

## Thanks

To the **Prowlarr** team and **LinuxServer.io** for the indexer hub and the
well-maintained image.

## Links

- Project: https://github.com/Prowlarr/Prowlarr
- Wiki: https://wiki.servarr.com/prowlarr
- Image: https://docs.linuxserver.io/images/docker-prowlarr
