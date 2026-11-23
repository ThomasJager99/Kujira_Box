# Snipe-IT

Snipe-IT is a **free, open-source IT asset management system** — it tracks your
hardware, software licenses, accessories and consumables, and who has what.
Think: every laptop, server, monitor and license key in one place, with
checkout/checkin to people, warranty and purchase dates, depreciation, audits,
QR/barcode labels, and a full history per asset. For a homelab it's a tidy way
to inventory your own gear; for a DevOps/IT portfolio it's a recognizable,
practical tool to have run.

Two services: **Snipe-IT** (PHP/Laravel app) and **MariaDB**.


<p align="center">
  <img src="../../../assets/logos/snipe-it.png" width="180">
</p>

<br>


## Official container images

- App: [`snipe/snipe-it`](https://hub.docker.com/r/snipe/snipe-it) — the
  maintainer's own image ([snipe/snipe-it](https://github.com/snipe/snipe-it)).
- DB: [`mariadb`](https://hub.docker.com/_/mariadb) (official).

Pin both (here `snipe/snipe-it:v8.8.4`, `mariadb:11`).

## Deployment Notes

- **`APP_KEY` is required and must be stable.** Generate it once
  (`echo "base64:$(openssl rand -base64 32)"`) and never change it — it encrypts
  stored data, so changing it later breaks decryption. If you start the app
  without one, it won't run.
- **`APP_URL`** should match how you actually reach it (your proxied `https://`
  host) so links and assets resolve correctly.
- **First run:** Snipe-IT runs DB migrations on first boot (give it a minute —
  the healthcheck has a 60s `start_period`), then the web UI walks you through a
  Pre-Flight setup wizard that creates the first admin user.
- **Data:** uploaded files + the generated app config live in `snipeit-data`;
  your actual inventory lives in the `snipeit-db` MariaDB volume. Back up
  **both**.
- **Email:** some features (sending credentials, alerts) want SMTP configured
  (extra `MAIL_*` env vars) — not required to run.
- **Port:** app on container `80`, published as `${SNIPEIT_PORT}`. Set
  `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse proxy.

### Security note

- MariaDB is on an **internal, no-egress network** (hardening entry 06) —
  reachable only by Snipe-IT.
- Bind address configurable (entry 07); put it behind reverse proxy + TLS
  (entry 09). It holds an inventory of your gear — don't leave it open.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# generate:  echo "base64:$(openssl rand -base64 32)"   (APP_KEY)
#            openssl rand -hex 24   (MYSQL_PASSWORD + ROOT)
# edit .env — paste APP_KEY, set APP_URL, LISTEN_ADDR
docker compose up -d
# open http://<host>:8095 → complete the Pre-Flight wizard → create admin → add assets
```

## Thanks

Thanks to [Snipe-IT / Grokability](https://github.com/snipe/snipe-it) for a
polished, genuinely free asset manager.

## Links

- Docs: https://snipe-it.readme.io/
- Docker install: https://snipe-it.readme.io/docs/docker
- Source: https://github.com/snipe/snipe-it
