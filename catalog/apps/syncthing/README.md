# Syncthing

**Syncthing** keeps folders in sync across your devices — continuously, in real
time, and directly peer-to-peer. There is no central server and no cloud: your
files travel straight between the machines you own, encrypted in transit, and
never sit on someone else's storage. You add a folder on one device, share it
with another by exchanging device IDs, and from then on changes propagate both
ways automatically, with file versioning so you can roll back mistakes. On a
homelab it's the self-hosted answer to Dropbox/Google Drive: sync your phone's
photos to the server, keep a documents folder mirrored across laptop and NAS, or
feed files into other services (a Paperless consume folder, a music library)
without any third party in the middle.


<p align="center">
  <img src="../../../assets/logos/syncthing.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `syncthing/syncthing`
- **Project:** https://github.com/syncthing/syncthing
- **Docs:** https://docs.syncthing.net/

## Deployment Notes

- **One container.** Single lightweight `syncthing` service (Go, Alpine-based).
- **Persistent state.** A named volume `syncthing_data` is mounted at
  `/var/syncthing` — it holds Syncthing's config and device keys as well as the
  default synced folders. **Your real data folders** should be added as extra
  bind mounts (see the commented example in the compose file) so they live on
  the host where you can back them up.
- **File ownership.** Runs under `PUID`/`PGID` (default `1000:1000`) — set these
  to match the user that owns your data folders, or synced files land with the
  wrong owner.
- **Web UI on port `8384`.** The image binds the UI to localhost by default, so
  `STGUIADDRESS=0.0.0.0:8384` is set to make the published port reachable. Keep
  the UI on your LAN/VPN — it controls what gets synced.
- **Sync ports.** `22000/tcp` and `22000/udp` carry the actual data transfer and
  must be reachable by your remote devices (forward them if syncing over the
  internet); `21027/udp` is local-network discovery.
- **Healthcheck** hits the no-auth `/rest/noauth/health` endpoint with the
  image's busybox `wget`.
- **Resource limits** (`mem_limit` / `mem_reservation` / `cpus`) and all
  environment-specific values are externalized to `.env`.

## Example Use

- Auto-backup your phone's camera roll to the homelab (pair with the Syncthing
  mobile app).
- Keep a `Documents` folder mirrored across laptop, desktop and server.
- Feed files into another service — e.g. a folder that Paperless-ngx consumes,
  or a music drop that Navidrome scans.
- Maintain an off-site copy by sharing a folder with a device at another
  location.

## Thanks

To the **Syncthing** project and its contributors for a private, open,
no-cloud-required way to sync files.

## Links

- Project: https://github.com/syncthing/syncthing
- Documentation: https://docs.syncthing.net/
- Getting started: https://docs.syncthing.net/intro/getting-started.html
