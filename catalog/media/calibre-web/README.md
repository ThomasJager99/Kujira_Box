# Calibre-Web

Calibre-Web is a clean, modern **web interface for browsing, reading and
managing an e-book library** — the one you've built with Calibre. It gives you
a bookshelf in the browser: cover grid, search and filters, an in-browser
reader (EPUB, PDF, comics, audiobooks), per-user accounts, OPDS feeds for
e-reader apps, and optional send-to-kindle and on-the-fly format conversion.

A quick distinction, because it trips people up: **Calibre** (the desktop app)
is what you *organise* your library with — it owns the `metadata.db`.
**Calibre-Web** is a lightweight server that *presents* that same library over
the web. Calibre-Web reads the library Calibre created; it doesn't replace
Calibre's management side. (If you want an e-book *reader* with its own library
model, that's Kavita/Komga — Calibre-Web is specifically the Calibre front-end.)


<p align="center">
  <img src="../../../assets/logos/calibre-web.svg" width="180">
</p>

<br>


## Official container image

[`lscr.io/linuxserver/calibre-web`](https://github.com/linuxserver/docker-calibre-web)
(LinuxServer.io), maintained multi-arch build. Upstream project:
[janeczku/calibre-web](https://github.com/janeczku/calibre-web).

> **Pinning:** LinuxServer publishes bare semver for this one (e.g. `0.6.27`) —
> pin it instead of `latest`.

## Deployment Notes

- **You need a Calibre library.** Point `BOOKS_PATH` at a folder that contains
  a Calibre library (i.e. a `metadata.db`). Don't have one? Point it at an
  empty directory and create the library from the web UI on first run, or build
  one first with desktop Calibre and drop it there.
- **First login:** `admin` / `admin123` — **change it immediately** (and set up
  per-user accounts under *Admin → Users*).
- **On first start** the UI asks for the "Location of Calibre database" — that's
  `/books` inside the container (your `BOOKS_PATH`).
- **Conversion / send-to-kindle:** these need Calibre's binaries, which the
  `DOCKER_MODS=linuxserver/mods:universal-calibre` line bakes in. Don't need
  them? Set `CALIBRE_WEB_MODS=` empty for a lighter image.
- **PUID / PGID** must match the owner of `BOOKS_PATH`, or Calibre-Web can't
  read/write the library.
- **Port:** UI on container `8083`, published as `${CALIBRE_WEB_PORT}`.
- **Exposure:** put it behind your reverse proxy with TLS if you want to read
  from outside the LAN; keep the LinuxServer hardening notes in the compose in
  mind (it runs its init as root inside the container).

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — pin version, set BOOKS_PATH to your Calibre library, PUID/PGID, TZ
docker compose up -d
# open http://<host>:8083 → set DB location to /books → log in admin/admin123 → change password
```

## Thanks

Thanks to [janeczku](https://github.com/janeczku/calibre-web) and contributors
for Calibre-Web, and to [LinuxServer.io](https://www.linuxserver.io/) for the
maintained image.

## Links

- Source: https://github.com/janeczku/calibre-web
- Wiki / setup: https://github.com/janeczku/calibre-web/wiki
- Container image: https://github.com/linuxserver/docker-calibre-web
