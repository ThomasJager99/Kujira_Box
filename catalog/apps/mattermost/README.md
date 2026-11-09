# Mattermost

Mattermost is a **self-hosted team chat platform** — the open-source
alternative to Slack/Discord/Teams. Channels, threads, DMs, file sharing,
search, slash commands, integrations and webhooks, plus mobile and desktop
apps — all running on your own box, with your messages in your own database.
The **Team Edition** used here is free and open-source; it's plenty for a
household, a friend group, or a small team.

This stack is two services: the **Mattermost app** and a **PostgreSQL**
database. It's also the rubric's showcase of the full hardened template on a
"normal" (non-LinuxServer) image — see the security notes below.


<p align="center">
  <img src="../../../assets/logos/mattermost.svg" width="180">
</p>

<br>


## Official container images

- App: [`mattermost/mattermost-team-edition`](https://hub.docker.com/r/mattermost/mattermost-team-edition)
- DB: [`postgres`](https://hub.docker.com/_/postgres) (official)

Project: [mattermost/mattermost](https://github.com/mattermost/mattermost).
Pin the app to a release number (here `12.0.2`) and Postgres to a major
(`16-alpine`).

## Deployment Notes

- **`MM_SITE_URL` must be your real public URL.** It's used for links, CORS and
  push notifications. Behind a reverse proxy that's `https://chat.example.com`,
  not `http://host:8065`. Set it before you invite anyone.
- **Database:** the app talks to Postgres over the internal network via
  `MM_SQLSETTINGS_DATASOURCE`, which is built from your `POSTGRES_*` values.
  Generate a strong `POSTGRES_PASSWORD` (`openssl rand -hex 24`).
- **First run:** open the UI and the very first account you create becomes the
  **System Admin**. Create yours immediately, then configure the server from
  *System Console*.
- **Port:** app on container `8065`, published as `${MATTERMOST_PORT}`. Put it
  behind your reverse proxy with TLS for real use.
- **Volumes:** config, data (uploads), logs, plugins and the search index are
  all named volumes — `mm-data` and `mm-db` are the ones that matter for
  backups.
- **Healthcheck** hits `/api/v4/system/ping`; the app can take up to a minute to
  come up on first boot (hence the generous `start_period`).

### Security: what's applied here (and why)

This is a good example of the hardening levels in practice:

- **Always-on (Level 1):** `no-new-privileges`, pinned images, memory/CPU and
  **PID limits** on both services, and a **frontend/backend network split** —
  the Postgres container sits on a `backend` network with `internal: true`, so
  it has **no route to the internet** and is unreachable from outside the stack.
- **Enabled where the image allows (Level 2):** the Mattermost image runs as a
  **non-root user** and binds a high port, so `cap_drop: [ALL]` is **active** on
  it — it needs no Linux capabilities at all.
- **Deliberately not forced (Level 3):** the Postgres image chowns its data dir
  as root during first-time init, so a blanket `cap_drop: [ALL]` on the DB can
  break `initdb`; it's left off there with a note on how to add it back
  selectively. `read_only` is likewise left off the app (it writes to its
  plugin dirs) — the volumes already contain the writable state.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — set MM_SITE_URL and a strong POSTGRES_PASSWORD
docker compose up -d
# open http://<host>:8065 → create the first account (becomes System Admin)
```

## Thanks

Thanks to the [Mattermost](https://github.com/mattermost/mattermost) team for a
genuinely capable open-source chat platform.

## Links

- Docs: https://docs.mattermost.com/
- Source: https://github.com/mattermost/mattermost
- Config via env: https://docs.mattermost.com/configure/configuration-settings.html
