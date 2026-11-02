# Grocy

Grocy is a self-hosted **groceries & household management** app — think of it
as a tiny ERP for the stuff behind your fridge door and kitchen cupboards. It
tracks your stock and its best-before dates (so it can warn you *before* food
expires), keeps shopping lists, stores recipes and works out what you still
need to buy for them, logs chores and household tasks, meters out battery
tracking, and gives you a barcode-driven "purchase / consume" workflow you can
drive from a phone. Everything is local — no cloud, no account with anyone but
you.

It's a single, self-contained container: nginx + PHP + an embedded **SQLite**
database all live inside the image, so there is no separate database service to
run. All of your data sits in one `/config` volume.


<p align="center">
  <img src="../../../assets/logos/grocy.svg" width="180">
</p>

<br>


## Official container image

[`lscr.io/linuxserver/grocy`](https://github.com/linuxserver/docker-grocy)
(LinuxServer.io) — the maintained, multi-arch (amd64 / arm64) build. Upstream
project: [grocy/grocy](https://github.com/grocy/grocy).

> **Pinning:** LinuxServer.io does **not** publish bare semver tags like
> `4.7.1`. Pin to a concrete build tag from the registry instead (e.g.
> `version-v4.7.1-ls256`) rather than leaving `latest`, so redeploys are
> reproducible. Current upstream release at time of writing: **v4.7.1**.

## Deployment Notes

- **No external database.** Grocy uses SQLite inside the container; the single
  `grocy-config` volume (`/config`) holds the DB, uploaded files and settings.
  Back *that* up and you've backed up everything.
- **PUID / PGID / TZ** follow the LinuxServer convention. Set `PUID`/`PGID` to
  the host user that should own `/config` (`id -u` / `id -g`), and `TZ` so the
  best-before and chore due dates land on the right day.
- **Port:** the UI is served on container port `80`, published on the host as
  `${GROCY_HTTP_PORT}` (default `9283`). Browse to `http://<host>:9283`.
- **Default login:** `admin` / `admin` — **change it immediately** on first
  sign-in (Settings → Users).
- **Exposure:** Grocy's own auth is basic. Don't expose it to the internet
  directly — put it behind your reverse proxy with TLS (and ideally your SSO /
  forward-auth), and keep the container on an internal network reachable only
  by the proxy.
- The LinuxServer image runs its s6 init as root inside the container, so the
  commented `read_only` / `cap_drop` hardening block is left opt-in rather than
  enabled by default.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — set GROCY_VERSION to a pinned tag, PUID/PGID, TZ
docker compose up -d
# open http://<host>:9283  → log in as admin/admin → change the password
```

Then: add your products and their locations, scan a barcode to "purchase" them
into stock, and let Grocy nag you about what's about to expire.

## Thanks

Thanks to the [Grocy](https://github.com/grocy/grocy) team and to
[LinuxServer.io](https://www.linuxserver.io/) for the maintained image.

## Links

- Project site: https://grocy.info/
- Source: https://github.com/grocy/grocy
- Container image: https://github.com/linuxserver/docker-grocy
- Demo: https://demo.grocy.info/
