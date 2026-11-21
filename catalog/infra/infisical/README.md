# Infisical

Infisical is a **self-hosted secrets management platform** — a central, access-
controlled home for the secrets your apps and infrastructure use: API keys,
database passwords, env vars, certificates. Instead of copying secrets into
`.env` files on every machine and hoping nobody commits one, you store them in
Infisical and pull them in at runtime via its CLI, SDKs, or
Kubernetes/agent/CI integrations — with versioning, per-environment scoping,
audit logs, and point-in-time recovery.

A distinction worth making: **Vaultwarden** (earlier in the series) is a
*personal password manager* for humans; **Infisical** is for *application &
infrastructure secrets* consumed by code and pipelines. Different jobs. For a
DevOps portfolio, running your own secrets manager is a strong piece.

Three services: **Infisical** + **PostgreSQL** (data) + **Redis** (cache/queues).


<p align="center">
  <img src="../../../assets/logos/infisical.svg" width="180">
</p>

<br>


## Official container images

- App: [`infisical/infisical`](https://hub.docker.com/r/infisical/infisical)
  ([Infisical/infisical](https://github.com/Infisical/infisical))
- DB: [`postgres`](https://hub.docker.com/_/postgres) · Cache:
  [`redis`](https://hub.docker.com/_/redis) (both official)

Pin all three. **Verify the exact app image tag on Docker Hub** — Infisical's
release tag here is `v0.167.14`; confirm the matching image tag before deploying.

## Deployment Notes

- **Generate the two core secrets** and keep them **stable** (they're used to
  encrypt everything — changing them orphans your data):
  - `ENCRYPTION_KEY` = `openssl rand -hex 16`
  - `AUTH_SECRET` = `openssl rand -base64 32`
- **`SITE_URL`** must match how you reach it (your proxied `https://` host) —
  used for links and the auth flow.
- **Datastores:** Postgres holds your secrets (encrypted at rest with
  `ENCRYPTION_KEY`); Redis handles caching/queues and needs no persistence here
  (it's configured as a pure cache). Back up the `infisical-db` volume — that's
  everything.
- **First run:** Infisical runs DB migrations on boot, then open the UI to
  create the first admin account and your first project/environment.
- **Email (optional):** invites and password resets need SMTP configured (extra
  `SMTP_*` env vars, see Infisical docs) — not required to run it solo.
- **Port:** app on container `8080`, published as `${INFISICAL_PORT}`. Set
  `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse proxy.

### Security note

This is a secrets manager, so treat it accordingly:

- Postgres **and** Redis are on an **internal, no-egress network** (hardening
  entry 06) — reachable only by Infisical.
- Always put it behind **reverse proxy + TLS** (entry 09); never expose a
  secrets store on plain HTTP.
- The `ENCRYPTION_KEY` / `AUTH_SECRET` / DB password in your `.env` are the exact
  case **hardening entry 14** is about — for anything serious, graduate them from
  a plaintext `.env` to Docker secrets or SOPS+age.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# generate:  openssl rand -hex 16  (ENCRYPTION_KEY)
#            openssl rand -base64 32  (AUTH_SECRET)
#            openssl rand -hex 24  (POSTGRES_PASSWORD)
# edit .env — paste all three, set SITE_URL, LISTEN_ADDR
docker compose up -d
# open http://<host>:8080 → create admin account → create a project → add secrets
```

## Thanks

Thanks to the [Infisical](https://github.com/Infisical/infisical) team for an
open-source, approachable secrets platform.

## Links

- Docs: https://infisical.com/docs/documentation/getting-started/overview
- Self-host guide: https://infisical.com/docs/self-hosting/overview
- Source: https://github.com/Infisical/infisical
