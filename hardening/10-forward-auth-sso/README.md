# 10 — Forward-auth / SSO: one login in front of everything

Entry 09 gave you one hardened front door with TLS. This entry puts a **lock**
on that door: a single sign-on layer that makes a user authenticate *before the
request ever reaches the app behind it*. It's how you safely expose a service
whose own login you don't fully trust — or that has no login at all.

## The risk: every app's own auth is a separate, uneven wall

Even behind a reverse proxy, each app still relies on **its own** login page.
That's a problem because:

- **Quality varies wildly.** Some apps have solid auth; some have a shared admin
  password; some (dashboards, metrics, that internal tool) have **no auth at
  all** and assume "it's on my network."
- **Pre-auth attack surface is exposed.** If an app has a login bug, an unauthed
  API endpoint, or a CVE in its login flow, anyone who can reach the page can
  reach the bug. The app's code is doing the gatekeeping.
- **No shared account control.** No single place to enforce MFA, disable a user
  everywhere at once, or require re-login.

**Forward authentication** fixes this by moving the gate *out* of the app and
*into* the proxy. The proxy asks an identity provider "is this person logged
in?" on every request. If not, they get bounced to one login page and never
touch the app until they pass. The app sees only authenticated traffic.

## How forward-auth works

```
            ┌──────────────┐   "is this user authed?"   ┌───────────────┐
  request ─►│ reverse proxy │ ─────────────────────────► │ Authelia /    │
   (443)    │ (NPM/Traefik) │ ◄───── yes/no ──────────── │ Authentik     │
            └──────┬───────┘                             └───────────────┘
                   │ only if authed
                   ▼
              the app  (never sees an unauthenticated request)
```

The proxy is configured so that, before proxying to the app, it makes a
sub-request to the identity provider's `/verify` (or `/api/authz/forward-auth`)
endpoint. Logged in → request passes through. Not → redirect to the SSO login,
then back. One login, one session, works across *all* your apps.

## The tools

| Tool | Weight | Good when |
|---|---|---|
| **Authelia** | light, config-file | you want MFA + forward-auth, minimal footprint |
| **Authentik** | heavier, full IdP (web UI) | you want OAuth2/OIDC/SAML/LDAP too, manage users in a UI |
| **tinyauth** | tiny | you just want a simple password/MFA gate |

You already built **Authentik** earlier in the service series — so you have a
full identity provider ready to be the brain here.

## The pattern (concept)

1. Run the identity provider (Authelia/Authentik) behind the same reverse proxy,
   on its own hostname (`auth.example.com`).
2. On a protected app's proxy host, add a **forward-auth** step pointing at the
   provider's verify endpoint. In Traefik it's a middleware; in NPM it's an
   "Access List" / advanced config calling `auth_request`; Caddy has
   `forward_auth`.
3. Define a policy: which users/groups may reach which app, and whether MFA is
   required.

Now hitting `grafana.example.com` first bounces you to `auth.example.com`, you
log in once (with MFA), and you're through to Grafana — and to every other
protected app — without another login.

## What to put behind it — and what NOT to

**Great behind forward-auth:**
- Dashboards and admin panels (Grafana, Portainer, metrics, that no-auth
  internal tool).
- Anything browser-based where you want MFA you don't otherwise have.

**Do NOT put behind forward-auth:**
- **Mail protocols (SMTP/IMAP) and other non-HTTP services** — forward-auth is
  an HTTP-layer thing; it can't gate a mail client's IMAP connection. Stalwart
  (today's service) authenticates mail with its own accounts — don't try to
  front its mail ports with SSO (its *admin web UI* behind SSO is fine).
- **APIs consumed by other programs** (webhooks, machine-to-machine) — a browser
  redirect breaks them. Use tokens/API keys for those, or bypass auth for
  specific paths.
- **Apps that already have robust auth + their own API/mobile apps** where the
  redirect would break the app clients (e.g. Nextcloud mobile) — use their
  native OIDC integration instead of a blunt forward-auth wall.

The rule: forward-auth is for **human, browser-facing** endpoints. Protocols and
machine clients authenticate their own way.

## How to verify

```bash
# an unauthenticated request to a protected app should redirect to the IdP:
curl -sI https://grafana.example.com | grep -i location
# -> location: https://auth.example.com/...     ✅ gated before the app

# a request with a valid session cookie passes through to the app (200):
curl -sI -H "Cookie: <your-session-cookie>" https://grafana.example.com | head -1
# -> HTTP/2 200
```

## Today's example

Stalwart Mail is the boundary case that teaches the limit: its **admin web UI**
is a perfect thing to put behind Authentik forward-auth, but its **mail ports**
(25/465/587/143/993) absolutely are not — IMAP clients can't do an OIDC browser
redirect, so mail keeps its own account auth. Same server, two different auth
models, chosen by *who's connecting and how*.

## Links

- Authelia: https://www.authelia.com/
- Authentik: https://goauthentik.io/
- Traefik ForwardAuth middleware: https://doc.traefik.io/traefik/middlewares/http/forwardauth/
- NGINX `auth_request`: https://nginx.org/en/docs/http/ngx_http_auth_request_module.html
