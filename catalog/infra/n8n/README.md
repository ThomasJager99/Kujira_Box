# n8n

**n8n** is a source-available workflow automation tool. Instead of wiring
services together with throwaway scripts, you build workflows visually on a
canvas — trigger nodes (a webhook, a schedule, an incoming email) connected to
action nodes (HTTP requests, database queries, message sending, branching
logic). It ships with hundreds of app integrations and, unlike most hosted
automation platforms, it can run entirely on your own hardware, so your data
and credentials never leave the box. It's a natural fit for a homelab: glue your
self-hosted services together, schedule maintenance jobs, or turn a webhook into
a chain of actions — all without a monthly bill or per-run quota.


<p align="center">
  <img src="../../../assets/logos/n8n.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `docker.n8n.io/n8nio/n8n`
- **Project:** https://github.com/n8n-io/n8n
- **Docs:** https://docs.n8n.io/hosting/

## Deployment Notes

- **One container.** Single `n8n` service. Defaults to an embedded **SQLite**
  database, which is fine for a personal instance; heavy/production use can be
  pointed at PostgreSQL via `DB_*` env vars (not enabled here to keep it simple).
- **Persistent state.** A named volume `n8n_data` is mounted at
  `/home/node/.n8n` — it holds the SQLite database, saved workflows and
  credentials. Back this up.
- **Single HTTP interface** on port `5678` (web editor + REST + webhooks).
- **Encryption key.** `N8N_ENCRYPTION_KEY` **must** be set once and kept stable;
  it encrypts stored credentials. Change it and existing credentials become
  unreadable. Generate with `openssl rand -hex 32`.
- **Address matters.** `N8N_HOST` / `N8N_PROTOCOL` / `WEBHOOK_URL` have to match
  how you actually reach n8n, otherwise generated webhook URLs point at the
  wrong place. Over plain HTTP on a trusted LAN, set `N8N_SECURE_COOKIE=false`
  or the login page won't load; prefer HTTPS behind a reverse proxy.
- **Healthcheck** hits the built-in `/healthz` endpoint using the image's
  busybox `wget`.
- **Resource limits** (`mem_limit` / `mem_reservation` / `cpus`) and all
  environment-specific values are externalized to `.env`.

## Example Use

- Schedule a nightly job that calls an API and drops the result into a database
  or a file.
- Turn an incoming webhook into a chain: parse → transform → notify (e.g. push a
  message to ntfy or Telegram).
- Watch an RSS feed / mailbox and auto-file or forward matching items.
- Sync data between two self-hosted apps on a timer.
- Build a small internal "no-code" API endpoint backed by a workflow.

## Thanks

To the **n8n** team and community for a genuinely powerful, self-hostable
automation platform.

## Links

- Project: https://github.com/n8n-io/n8n
- Hosting docs: https://docs.n8n.io/hosting/
- Environment variables: https://docs.n8n.io/hosting/configuration/environment-variables/
