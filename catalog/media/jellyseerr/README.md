# Jellyseerr

**Jellyseerr** is a request-and-discovery front end for your media server. It's
a fork of Overseerr built to work natively with **Jellyfin** and **Emby** (as
well as Plex). Users browse a polished catalogue of movies and TV shows, and
when they want something that isn't in your library they click **Request** —
Jellyseerr then hands that request to your **Sonarr/Radarr** to download and
organise it, tracks its status, and notifies the requester when it's available.
It brings user accounts, approval workflows and notifications on top of your
existing stack, so you can let family or friends "order" content without giving
them access to the plumbing. For a homelab it's the friendly storefront that
sits in front of Jellyfin and the *arr apps.


<p align="center">
  <img src="../../../assets/logos/jellyseerr.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `fallenbagel/jellyseerr`
- **Project:** https://github.com/fallenbagel/jellyseerr
- **Docs:** https://docs.jellyseerr.dev/

> **Heads-up — rebranding.** The project is being renamed **Jellyseerr → Seerr**
> (repository `seerr-team/seerr`; latest release `v3.5.0` at the time of
> writing). The `fallenbagel/jellyseerr` image is still published; pin the tag
> you want and watch for the image name changing to the new Seerr one.

## Deployment Notes

- **One container.** Single `jellyseerr` service — no separate database (it uses
  an embedded SQLite file under its config dir).
- **Single HTTP interface** on port `5055`.
- **Persistent state.** Named volume `jellyseerr_config` (`/app/config`) holds
  the SQLite database and all settings. Back it up.
- **First run.** On first launch you sign in with your Jellyfin/Emby/Plex
  account, then connect Jellyseerr to Jellyfin and to your Sonarr/Radarr
  instances — all from the setup wizard in the web UI (no secrets in `.env`).
- **Pinned version.** `.env` lets you pin `JELLYSEERR_VERSION`; given the
  rebrand, confirm the current tag in the registry before deploying.
- **Healthcheck** fetches `/api/v1/status` with the image's busybox `wget`.
- **Optional hardening** (commented in compose): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.
- Single container with no database, so the upgraded template's
  frontend/backend network isolation isn't needed here.

## Example Use

- Let household users request new movies/shows without touching Jellyfin admin.
- Auto-route approved requests to Sonarr/Radarr for download.
- Approve/deny requests and set per-user quotas.
- Notify requesters (e.g. via your ntfy server) when their title is ready.

## Thanks

To **fallenbagel** and the Jellyseerr/Seerr contributors for bringing
Overseerr's experience to Jellyfin and Emby.

## Links

- Project: https://github.com/fallenbagel/jellyseerr
- Documentation: https://docs.jellyseerr.dev/
- Overseerr (upstream): https://overseerr.dev/
