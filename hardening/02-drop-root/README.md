# 02 — Drop root: run containers as a non-root user

The single most valuable line in the baseline block from entry 01. If you do
only one thing from this whole rubric, do this one.

## The risk: "root in the container" is closer to "root on your host" than you think

By default, the process inside a container runs as **root (UID 0)** — and that
UID 0 is the *same* UID 0 as your host's root, because containers share the
host kernel (entry 01). Docker throws some fences around it (namespaces,
default-dropped capabilities), but those fences are exactly what an escape or a
misconfiguration chips away at. The pattern is always the same:

> container root **+ one extra mistake** (a mounted host path, a kernel CVE, a
> capability you added, a bind-mounted socket) = **root on your machine**.

Running as a normal, unprivileged user removes the *first half* of that
equation. Now an escape lands the attacker as UID 1000 with nothing — not root.
It also limits the damage *inside* the container: a compromised process can't
rewrite system files, install packages, or bind low ports.

## The fix: tell the container who to be

### Option A — set `user:` in compose (works for most images)

```yaml
services:
  app:
    image: example/app:1.2.3
    user: "1000:1000"        # UID:GID — an unprivileged user
```

Pick a UID/GID that owns any volumes the app writes to. On your host:

```bash
id -u   # -> 1000   (your user's UID)
id -g   # -> 1000
```

If the app writes to a **named volume**, pre-create ownership or let the app's
entrypoint chown it; if it writes to a **bind mount**, make sure that host
directory is owned by the same UID (`chown -R 1000:1000 ./data`).

### Option B — images built to run non-root (the clean way)

Good images already define a non-root user in their Dockerfile:

```dockerfile
RUN adduser --system --uid 1000 app
USER app
```

Many official images ship this (e.g. `postgres` runs as `postgres`, `node` has
a `node` user). You don't need `user:` for those — but setting it never hurts
as an explicit guarantee.

### The LinuxServer.io exception (PUID/PGID)

LinuxServer images (`lscr.io/linuxserver/*` — Grocy, Sonarr, Radarr…) start
their **s6 init as root inside the container** and then drop the *app* to the
`PUID`/`PGID` you give. So you *don't* set `user:` on those — you set
`PUID`/`PGID` env instead, and accept that the init shim is root. For those,
lean on the other defenses (no socket, no `privileged`, network isolation,
reverse proxy). Knowing *which* model an image uses is half the battle.

## How to verify

```bash
# Who is the main process actually running as?
docker exec <container> id
# want: uid=1000(...) gid=1000(...)   — NOT uid=0(root)

# Double-check from the host side:
docker top <container>
# the USER column should not be root
```

If it says `uid=0`, either the image forces root (check its docs) or your
`user:` line didn't take — look for an entrypoint that re-escalates.

## When an app won't start as non-root

Common and fixable:

- **"permission denied" writing data** → the volume is owned by root. `chown`
  the bind-mount dir, or use a named volume the entrypoint can set up.
- **"bind: permission denied" on port 80/443** → non-root can't bind ports
  <1024. Publish a high port and let your reverse proxy front it
  (`ports: ["8080:8080"]`), or add the `NET_BIND_SERVICE` capability (entry 03).

Loosen one thing, not everything.

## Today's real example

Woodpecker CI (today's service) is the *counter*-example that proves the rule:
its **agent** can't drop the socket and often runs privileged, because its job
is to start other containers. That's exactly why you run the agent on a
throwaway CI host — when you *can't* drop privilege, you isolate the host
instead. Everything that *can* run as non-root, should.

## Links

- Docker `user` / userns: https://docs.docker.com/engine/security/userns-remap/
- Compose `user`: https://docs.docker.com/reference/compose-file/services/#user
- OWASP Docker Security Cheat Sheet (Rule 2): https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html
