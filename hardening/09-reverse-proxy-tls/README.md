# 09 — Reverse proxy + TLS in front of everything

The last few entries kept telling you to "put it behind your reverse proxy."
This is the entry that explains why that one piece of infrastructure is the
backbone of a safe self-hosted setup — and what it actually buys you.

## The risk: a dozen apps, each exposed, each on its own

Without a reverse proxy, every service you run tends to get its own published
port on the LAN: `:5090` here, `:8065` there, `:8083` somewhere else. That means:

- **No encryption** — plain HTTP on every one. Anyone on the network path (a
  compromised device, an untrusted Wi-Fi, your ISP if it's ever port-forwarded)
  can read passwords, sessions and data in the clear.
- **A wide, uneven attack surface** — a dozen app ports open, each with its own
  login page, each as strong (or weak) as that app's own auth, each a separate
  thing to remember and lock down.
- **No central control** — no single place to add authentication, rate limiting,
  IP allowlists, security headers, or to simply turn something off.

A **reverse proxy** collapses all of that into **one front door**. Every app
sits behind it on an internal network; the proxy is the only thing exposed, it
terminates TLS, and it routes by hostname to the right app.

## What a reverse proxy gives you

```
                                   ┌─ excalidraw  (no published port)
  Internet / LAN ──► reverse ─────┼─ mattermost  (no published port)
      (443 only)     proxy  (TLS) └─ grocy       (no published port)
```

- **TLS everywhere, in one place.** The proxy gets the certificates (Let's
  Encrypt, automatic renewal) and serves every app over HTTPS. Your apps don't
  each need to know about certs.
- **One exposed port (443).** Everything else moves to `127.0.0.1` or an
  internal docker network with **no** `ports:` (entries 06–07). The LAN sees
  only the proxy.
- **Routing by hostname.** `chat.example.com`, `draw.example.com`,
  `grocy.example.com` all hit :443 and the proxy sends each to the right
  container by name over the shared docker network.
- **A single choke point for policy** — add auth (entry 10), rate limits,
  security headers (HSTS, X-Frame-Options, CSP), IP allowlists, or block a path,
  all in one config instead of twelve.

## The options

| Proxy | Style | Good when |
|---|---|---|
| **Nginx Proxy Manager** | Web UI, click-to-configure, built-in Let's Encrypt | you want a GUI; you already run it (you do) |
| **Traefik** | Auto-discovers containers via labels, dynamic | lots of containers coming and going |
| **Caddy** | Tiny config file, automatic HTTPS by default | you like config-as-text, minimalism |
| **Nginx** (raw) | Full manual control | you want to hand-write everything |

You already run **Nginx Proxy Manager** — so the pattern below is the one to
standardize on.

## The pattern (with Nginx Proxy Manager)

1. Put each app on a **shared docker network** with NPM, and give it **no
   published port** (or bind `127.0.0.1` only):

   ```yaml
   services:
     excalidraw:
       image: excalidraw/excalidraw:latest
       networks: [proxy]        # shared with NPM; NO ports: line
   networks:
     proxy:
       external: true           # the network NPM lives on
   ```

2. In NPM → **Proxy Hosts → Add**:
   - Domain: `draw.example.com`
   - Forward to: `excalidraw` : `80` (the container name + internal port)
   - **SSL tab:** request a Let's Encrypt cert, force HTTPS, enable HSTS.

3. Point DNS (`draw.example.com`) at the proxy. Done — the app is HTTPS-only and
   its own port is never exposed.

## TLS specifics worth knowing

- **Let's Encrypt** = free, auto-renewing certs. Needs a real domain and either
  port 80 reachable (HTTP-01) or DNS API access (DNS-01, works for internal-only
  hosts).
- **Internal-only / no public domain?** Use **DNS-01** with your DNS provider's
  API to get real certs for `*.internal.example.com` without exposing anything,
  or run an internal CA (mkcert/step-ca) and trust it on your devices.
- **Always force HTTP→HTTPS** and enable **HSTS** so browsers refuse to downgrade.
- Terminating TLS at the proxy and talking plain HTTP to the container **over an
  internal docker network** is fine — that traffic never leaves the host.

## How to verify

```bash
# the app's own port is NOT exposed on the LAN (only the proxy's 443 is):
nmap -p 443,5090,8065,8083 <host>        # 443 open; app ports closed/filtered

# TLS actually served + redirect works:
curl -sI http://draw.example.com | grep -i location     # -> https://...
curl -sI https://draw.example.com | grep -i strict-transport-security   # HSTS present

# cert is valid (not the app's self-signed default):
echo | openssl s_client -connect draw.example.com:443 2>/dev/null | openssl x509 -noout -issuer
# -> issuer = Let's Encrypt (or your CA)
```

## Today's example

Excalidraw (today's service) is the textbook case: a stateless static app you
do **not** want to publish on its own port. Set `LISTEN_ADDR=127.0.0.1` (or drop
`ports:` entirely and join NPM's network), add a proxy host `draw.example.com`
with a Let's Encrypt cert, and it's HTTPS-only with zero LAN exposure. Multiply
that by every service and the whole homelab presents **one** hardened front
door instead of a dozen.

## Links

- Nginx Proxy Manager: https://nginxproxymanager.com/
- Caddy automatic HTTPS: https://caddyserver.com/docs/automatic-https
- Traefik: https://doc.traefik.io/traefik/
- Let's Encrypt / HSTS: https://letsencrypt.org/docs/ · https://developer.mozilla.org/docs/Web/HTTP/Headers/Strict-Transport-Security
