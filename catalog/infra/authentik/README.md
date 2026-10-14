# authentik

**authentik** is a self-hosted identity provider (IdP) and single-sign-on (SSO)
platform — the open-source alternative to Okta / Auth0 / Keycloak. You run it
once, connect your apps to it, and it becomes the single place where users log
in. It speaks all the standard protocols — **OAuth2 / OpenID Connect**, **SAML**,
**LDAP**, and **forward/proxy auth** — so almost anything can authenticate
against it, including apps that have no native SSO (you put authentik's proxy in
front of them). On top of that it gives you multi-factor auth, flexible login
flows, user self-service, and policies. For a homelab it's the "one login for
everything" upgrade: stop managing a separate password in every service and put
authentik in front of them all.


<p align="center">
  <img src="../../../assets/icons/app-window.svg" width="180">
</p>

<br>



## Official container image

- **Image:** `ghcr.io/goauthentik/server` (run as `server` and as `worker`)
- **Project:** https://github.com/goauthentik/authentik
- **Docs:** https://docs.goauthentik.io/

## Deployment Notes

- **Four containers:**
  - `server` — the web UI, API and auth endpoints; HTTP on `9000`, HTTPS on `9443`.
  - `worker` — background tasks (emails, outpost management, scheduled jobs);
    same image, `command: worker`.
  - `postgresql` — **PostgreSQL 16**, the database.
  - `redis` — cache and task queue.
- **Startup order.** `server` and `worker` wait for `postgresql` and `redis` to
  be `service_healthy`.
- **Secret key.** `AUTHENTIK_SECRET_KEY` signs sessions/tokens — set a strong
  value (`openssl rand -base64 60`) once and keep it stable.
- **First run.** After the stack is up, open `http://<host>:9000/if/flow/initial-setup/`
  to create the first admin (`akadmin`) and set its password.
- **Persistent state.** Named volumes: `authentik_media` (`/media`, includes
  uploaded certs/branding), `authentik_templates` (`/templates`), `db_data`
  (PostgreSQL) and `redis_data`. Back up `db_data` + `authentik_media`.
- **Healthchecks:** server/worker use the built-in `ak healthcheck`; postgres
  uses `pg_isready`; redis uses `redis-cli ping`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

### House-style upgrades used here

- **Pinned versions.** authentik uses **date-based** versions (e.g. `2026.8.3`) —
  pin exactly and upgrade deliberately (read release notes; migrations run on
  upgrade). Postgres/Redis pinned too.
- **Network isolation.** `frontend` (published ports + internet) and `backend`
  (`internal: true`, no internet). **postgresql and redis live on `backend`
  only** — reachable by server/worker, invisible outside the stack, no egress.
  Server and worker are on both (the worker needs egress for email/external
  sources).
- **Optional hardening** (commented on `server`): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing.
- **Docker socket is opt-in.** The worker can manage outposts via the Docker
  socket, but that's host-root power — it's left **commented**; enable only if
  you need it, ideally behind a socket-proxy.
- **Secrets.** `.env` plaintext is fine for homelab; for production move to
  Docker secrets or SOPS + age.

## Example Use

- One login (with MFA) for all your self-hosted apps via OIDC/SAML.
- Protect an app that has no auth of its own using authentik's proxy provider.
- Provide an LDAP endpoint for older software that only speaks LDAP.
- Give users self-service password reset and enrollment.

## Thanks

To the **authentik** team for a powerful, fully self-hostable identity provider.

## Links

- Project: https://github.com/goauthentik/authentik
- Documentation: https://docs.goauthentik.io/
- Docker install: https://docs.goauthentik.io/docs/install-config/install/docker-compose
