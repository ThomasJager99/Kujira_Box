# PairDrop

PairDrop is a **local, browser-based way to send files between any two
devices** — think AirDrop, but open-source, cross-platform, and self-hosted.
Open the page on your phone and your laptop, they see each other, you drag a
file across. No app to install, no account, nothing uploaded to a cloud:
PairDrop is only a **signaling server** that helps two browsers find each
other, after which the transfer goes **peer-to-peer over WebRTC** and never
touches your server.

Because of that, it's basically **stateless** — there's no database and no data
volume to back up. A tiny, delightful utility for a household or an office LAN.


<p align="center">
  <img src="../../../assets/logos/pairdrop.png" width="180">
</p>

<br>


## Official container image

[`lscr.io/linuxserver/pairdrop`](https://github.com/linuxserver/docker-pairdrop)
(LinuxServer.io), the maintained multi-arch build. Upstream project:
[schlagmichdoch/PairDrop](https://github.com/schlagmichdoch/PairDrop).

> **Pinning:** LinuxServer.io doesn't publish bare semver tags — pin to a build
> tag like `version-v1.11.2` (current upstream release) instead of `latest`.

## Deployment Notes

- **How devices pair:** PairDrop groups devices by their **public IP**. On one
  LAN that just works — everyone shares the router's public IP, so they all
  land in the same room automatically.
- **Behind a reverse proxy — the one gotcha:** the proxy *must* forward the
  real client IP (`X-Forwarded-For` / `X-Real-IP`). If it doesn't, every device
  appears to come from the proxy's IP, so they either all pair together
  (wrong) or none do. Set this in your Nginx/NPM/Traefik config.
- **Different networks / strict firewalls:** direct WebRTC may be blocked. Turn
  on `PAIRDROP_WS_FALLBACK=true` to relay the transfer over WebSocket through
  the server as a fallback. (Then the bytes *do* pass through the server, so
  only enable it if you need it.)
- **Stateless:** no volume, no DB. Updating is just pulling a new image.
- **Port:** UI on container `3000`, published as `${PAIRDROP_PORT}`.
- **Exposure:** fine on the LAN as-is; if you expose it to the internet, put it
  behind TLS + your reverse proxy and consider `PAIRDROP_RATE_LIMIT=true`.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — pin PAIRDROP_VERSION, set PUID/PGID, TZ
docker compose up -d
# open http://<host>:3000 on two devices on the same network → drag a file across
```

## Thanks

Thanks to [schlagmichdoch](https://github.com/schlagmichdoch/PairDrop) for
PairDrop (itself a fork/evolution of Snapdrop), and to
[LinuxServer.io](https://www.linuxserver.io/) for the image.

## Links

- Project: https://github.com/schlagmichdoch/PairDrop
- Public instance: https://pairdrop.net/
- Container image: https://github.com/linuxserver/docker-pairdrop
