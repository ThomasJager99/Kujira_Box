# 14 — Secrets: beyond the plaintext `.env`

This entry opens **block B — not trusting blindly**, and it starts at home: the
secrets *you* put into your own stacks. Every compose file in this series reads
passwords and keys from a `.env` file. That's the pragmatic homelab default and
it's fine — until it isn't. This is about knowing *where a plaintext `.env`
leaks* and what the next steps up look like when a secret actually matters.

## The risk: a plaintext `.env` leaks in ordinary ways

A `.env` full of real passwords sitting next to your compose file is readable
plaintext. It leaks through completely mundane paths:

- **Git.** The #1 way secrets escape. One `git add .`, one forgotten `.env` not
  in `.gitignore`, and your DB password is in history — public if the repo is.
  (This whole series ships `.env.example`, never `.env`, for exactly this
  reason.) And git history is forever: deleting the file later doesn't remove it
  from past commits.
- **Backups & snapshots.** Your `.env` rides along into every backup, often less
  guarded than the live host.
- **`docker inspect` / the process list.** Secrets passed as **environment
  variables** show up in `docker inspect <container>`, in `/proc/<pid>/environ`,
  and sometimes in logs/crash dumps. Anyone who can run `docker inspect` (i.e.
  anyone in the docker group — entry 08) reads them.
- **Shoulder-surfing & sync.** A `.env` copied to a laptop, synced to a cloud
  drive, or shown on a screen-share.

For a solo homelab behind a firewall, `.env` is an *acceptable* risk. The point
isn't "never use it" — it's knowing it's the floor, not the ceiling, and
stepping up when the secret is worth it.

## Step 1 — get the basics right (do this always)

- **`.gitignore` your `.env`.** Non-negotiable. Commit only `.env.example` with
  placeholders (this series' convention).
- **Restrict permissions:** `chmod 600 .env` so only the owner reads it.
- **Never bake secrets into the image** (no `ENV PASSWORD=` in a Dockerfile —
  it's in every layer, pullable by anyone).
- **Generate, don't reuse:** `openssl rand -hex 24` per secret; unique per
  service.

## Step 2 — Docker secrets (prefer files over env vars)

Docker's **secrets** mechanism mounts a secret as a **file** inside the
container (`/run/secrets/<name>`) instead of an env var — which keeps it out of
`docker inspect` and the environment:

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      # many images accept a *_FILE variant that reads from a file:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt     # file on disk, not in the compose/env
```

Lots of official images (postgres, mysql, …) support the `*_FILE` convention.
This alone fixes the "secrets visible in `docker inspect`/environ" leak. The
file still lives on disk, so it's a step, not the finish line.

## Step 3 — encrypt secrets at rest with SOPS + age

When you *do* want secrets in git (GitOps, sharing a repo across machines)
without storing them in plaintext, encrypt them with **SOPS** +
**age**:

```bash
# one-time: make an age keypair
age-keygen -o key.txt          # prints a public key "age1..."

# encrypt a secrets file (values encrypted, keys stay readable):
sops --encrypt --age age1xxxx... secrets.env > secrets.enc.env
# -> safe to commit; only someone with the private key can decrypt

# decrypt at deploy time:
sops --decrypt secrets.enc.env > .env    # then docker compose up
```

SOPS encrypts the **values** (leaving keys/structure readable for diffs), so an
encrypted file is git-safe. The private age key lives outside the repo (on the
host, in your password manager). This is the sweet spot for a homelab that
version-controls its stacks.

## Step 4 — a secrets manager (when it grows up)

Past a certain scale you stop storing secrets in files at all and pull them at
runtime from a **secrets manager** — HashiCorp Vault / OpenBao, or **Infisical**
(today's service). Apps/CI authenticate and fetch secrets on demand, with
rotation, audit logs and per-environment scoping. Overkill for two stacks;
exactly right once you have many apps, machines, or people.

## How to verify

```bash
# is a real .env about to be committed? (should show nothing/ignored)
git check-ignore .env && echo "ignored ✅" || echo "NOT ignored — fix .gitignore"

# are secrets leaking via the environment? (env-var secrets show here)
docker inspect <container> --format '{{ range .Config.Env }}{{ println . }}{{ end }}' | grep -i -E 'pass|secret|key|token'
# with Docker secrets (*_FILE), you'll see the PATH, not the value ✅

# permissions locked down?
stat -c '%a' .env     # -> 600
```

## Today's example

Infisical (today's service) *is* step 4 — a secrets manager you can run
yourself. But notice the irony the README points out: to bootstrap it you still
set `ENCRYPTION_KEY`, `AUTH_SECRET` and a DB password in a plaintext `.env`.
That's the ladder in one picture: the bootstrap secrets want at least Step 2/3
treatment, and then Infisical holds everything *above* it for your other apps.
Match the rung to the value of the secret.

## Links

- Docker secrets (Compose): https://docs.docker.com/compose/how-tos/use-secrets/
- SOPS: https://github.com/getsops/sops
- age: https://github.com/FiloSottile/age
- OpenBao (open-source Vault): https://openbao.org/
