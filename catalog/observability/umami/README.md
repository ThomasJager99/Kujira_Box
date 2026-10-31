# Umami

**Umami** is a simple, fast, **privacy-friendly** web analytics tool — a
self-hosted alternative to Google Analytics. You add a tiny script to your
website and Umami shows you the essentials (visitors, page views, referrers,
countries, devices, events) in a clean dashboard — **without cookies, without
tracking individuals, and without shipping your visitors' data to a third
party**. Because it collects no personal data, it's GDPR/PECR-friendly and
usually needs no cookie banner. It's lightweight, supports multiple websites and
team accounts, and all the data stays in your own database. For a homelab it's
the "know how my sites are doing, privately" tool.


<p align="center">
  <img src="../../../assets/logos/umami-light.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `ghcr.io/umami-software/umami` (PostgreSQL variant)
- **Project:** https://github.com/umami-software/umami
- **Docs:** https://umami.is/docs

## Deployment Notes

- **Two containers:** `umami` (the app, single HTTP interface on `3000`) and
  `db` (**PostgreSQL 16**). The image comes in `postgresql-*` and `mysql-*`
  flavours — this uses PostgreSQL.
- **Startup order.** The app waits for `db` to be `service_healthy`.
- **Database URL.** The app connects via `DATABASE_URL` pointing at the `db`
  service; it runs its own schema migrations on first start.
- **App secret.** `APP_SECRET` signs auth tokens — set a strong value
  (`openssl rand -hex 32`) and keep it stable.
- **Persistent state.** Named volume `db_data` (PostgreSQL). Back it up with
  `pg_dump` — that's where all your analytics live.
- **First login.** Default credentials are **`admin` / `umami`** — change the
  password immediately after first login.
- **Pinned version.** `.env` pins `UMAMI_VERSION` (image tag like
  `postgresql-v3.4.1`) and `POSTGRES_VERSION`. Verified latest at build time:
  `v3.4.1`; confirm the exact tag in the registry.
- **Healthcheck** fetches `/api/heartbeat` with the image's busybox `wget`;
  `db` uses `pg_isready`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

### House-style upgrades used here

- **Pinned versions** (app tag + Postgres).
- **Network isolation.** `frontend` (published port + internet) and `backend`
  (`internal: true`). The **database is on `backend` only** — reachable by the
  app, invisible outside the stack, no internet egress.
- **Optional hardening** (commented on the app): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing.
- **Secrets.** `.env` plaintext is fine for homelab; for production move to
  Docker secrets or SOPS + age.

## Example Use

- Track visits to your sites/blog without Google Analytics or cookie banners.
- See referrers, countries, devices and top pages at a glance.
- Track custom events (button clicks, sign-ups) privately.
- Run analytics for several sites from one dashboard.

## Thanks

To the **Umami** team for a clean, privacy-respecting, self-hostable analytics
platform.

## Links

- Project: https://github.com/umami-software/umami
- Documentation: https://umami.is/docs
- Docker install: https://umami.is/docs/install
