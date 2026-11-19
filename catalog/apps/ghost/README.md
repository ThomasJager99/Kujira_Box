# Ghost

Ghost is a **modern publishing platform** — a blog, newsletter and membership
site rolled into one. It has a genuinely pleasant writing editor, clean built-in
themes, native email newsletters, and paid subscriptions/memberships if you ever
want them. It's what a lot of independent writers use instead of WordPress or
Substack. Self-hosting it means your writing lives on *your* domain, in *your*
database — no platform between you and your readers.

(Handy if you're reviving a dev blog: your homelab write-ups could live here
instead of a third-party site.)

Two services: **Ghost** and **MySQL 8** (the database Ghost officially supports
in production).


<p align="center">
  <img src="../../../assets/logos/ghost-platform-icon.svg" width="180">
</p>

<br>


## Official container images

- App: [`ghost`](https://hub.docker.com/_/ghost) (official)
- DB: [`mysql`](https://hub.docker.com/_/mysql) (official, version **8**)

Pin the app to a major (`6-alpine`) and MySQL to `8.0`.

## Deployment Notes

- **`url` is critical.** Ghost redirects every request to its configured `url`,
  so it must match exactly how readers reach the site (`https://blog.example.com`
  behind your proxy). Get this wrong and you'll get redirect loops or assets
  loading from the wrong host.
- **MySQL 8 specifically.** Ghost 5/6 support MySQL 8; don't substitute MariaDB
  for the production database — use the `mysql:8.0` image here.
- **First run:** once it's up, go to `https://<your-url>/ghost/` — the first
  visit walks you through creating the owner account and naming the site.
- **Data:** your themes, images, and config live in `ghost-content`; your posts
  and settings live in the MySQL volume `ghost-db`. Back up **both**.
- **Email:** newsletters need an external mail provider (Mailgun is Ghost's
  first-class option) configured in Ghost's settings — not required just to run
  the site.
- **Port:** app on container `2368`, published as `${GHOST_PORT}`. Set
  `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse proxy.

### Security note

- MySQL is on an **internal, no-egress network** (hardening entry 06) —
  reachable only by Ghost.
- Bind address is configurable (entry 07); a public blog is a prime case for
  reverse proxy + TLS (entry 09) with the app port kept off the LAN.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# generate 2 strong passwords:  openssl rand -hex 24   (MYSQL_PASSWORD + ROOT)
# edit .env — set GHOST_URL to your real URL, LISTEN_ADDR
docker compose up -d
# open https://<your-url>/ghost/ → create the owner account → start writing
```

## Thanks

Thanks to the [Ghost](https://github.com/TryGhost/Ghost) team and Ghost
Foundation for a first-class open-source publishing platform.

## Links

- Docs: https://ghost.org/docs/
- Config reference: https://ghost.org/docs/config/
- Source: https://github.com/TryGhost/Ghost
