# Nginx Proxy Manager

**Nginx Proxy Manager** (NPM) puts a friendly web UI in front of Nginx so you
can expose your self-hosted services under real hostnames with valid HTTPS —
without hand-editing Nginx config files. You point your domains at the box, then
in the UI create a "Proxy Host" that maps `app.example.com` to an internal
service (`http://127.0.0.1:5678`, another container, whatever), and NPM requests
and auto-renews a free **Let's Encrypt** certificate for it. It also handles
access lists, custom SSL, redirects and streams. For a homelab it's the piece
that turns a pile of `IP:port` services into clean, encrypted URLs — and it's
what several other tools here (Vaultwarden, n8n, etc.) want in front of them,
since their clients expect HTTPS.


<p align="center">
  <img src="../../../assets/logos/nginx-proxy-manager.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `jc21/nginx-proxy-manager`
- **Project:** https://github.com/NginxProxyManager/nginx-proxy-manager
- **Docs:** https://nginxproxymanager.com/

## Deployment Notes

- **One container.** Single `npm` service bundling Nginx + the management app.
  Defaults to an embedded **SQLite** database (kept in `/data`), which is fine
  for a homelab; a MySQL/MariaDB backend can be configured via `DB_MYSQL_*` env
  vars if you prefer.
- **Three ports:**
  - `80` and `443` — the public HTTP/HTTPS that your proxied sites are served
    on; port `80` must be reachable for Let's Encrypt's HTTP validation.
  - `81` — the **admin UI**. Keep this restricted to your LAN/VPN; never expose
    it to the internet.
- **Persistent state.** Two named volumes: `npm_data` (`/data` — config,
  database, custom Nginx snippets) and `npm_letsencrypt` (`/etc/letsencrypt` —
  issued certificates). Back both up.
- **First login.** Default credentials are `admin@example.com` / `changeme` —
  the UI forces you to change the email and password immediately on first login.
- **Healthcheck.** The image already ships a `/bin/check-health` script; this
  stack adds an explicit `curl` to the admin port `81` so an unhealthy UI is
  visible in `docker ps`.
- **Resource limits** (`mem_limit` / `mem_reservation` / `cpus`) and all
  environment-specific values are externalized to `.env`.

## Example Use

- Give every homelab service a clean hostname with automatic HTTPS
  (`vault.example.com`, `photos.example.com`, …).
- Terminate TLS in one place instead of configuring certificates per app.
- Put an access list / basic auth in front of a service that has no auth of its
  own.
- Redirect old URLs, or forward a raw TCP/UDP stream to an internal host.

## Thanks

To **jc21** and the Nginx Proxy Manager contributors for making Nginx +
Let's Encrypt approachable through a clean UI.

## Links

- Project: https://github.com/NginxProxyManager/nginx-proxy-manager
- Documentation: https://nginxproxymanager.com/
- Setup guide: https://nginxproxymanager.com/setup/
