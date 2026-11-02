# 01 — Threat model & baseline hardening

The first entry of the rubric does two things: it's honest about **what a
container actually protects you from**, and it gives you the **first round of
defense** as a single block you can paste into any stack. Every later entry
drills into one line of that block.

## The risk: a container is a boundary, not a guarantee

It's tempting to think "it's in Docker, so it can't touch my system." That's
*mostly* true and it's why containers are genuinely good for running other
people's code — but it's not a wall, it's a fence. Two things to keep straight:

**1. Containers share the host kernel.** Unlike a VM (which has its own kernel),
every container talks to the *same* Linux kernel as your host. A kernel bug, or
a privilege you hand the container, can become a way out ("container escape").
You widen that door yourself whenever you use:

- `privileged: true` — basically turns the fence off
- a mounted `/var/run/docker.sock` — the Docker socket **is** root on the host
- `network_mode: host` — the container sees your whole host network
- host path bind-mounts — the container can read/write those host files
- running as **root inside the container** (the default for many images)

**2. The scarier risk usually isn't escape at all.** A malicious image doesn't
need to break *out* to hurt you — it can simply abuse **the data and access you
handed it**: read the files you mounted, steal the secrets in its environment,
and phone home over its normal network access. No escape required.

So the job splits in two, and this whole rubric follows that split:

- **Shrink what a container is *allowed* to do** (block A — starts below).
- **Don't run a bad image in the first place** (block B — vetting).
- **Test anything truly untrusted in a real VM, not a container** (block C).

## The fix: your baseline hardening block

This is the "first round" — a safe default you can add to almost any service.
It assumes the app runs fine unprivileged (most do). See
[`baseline-hardened.example.yml`](baseline-hardened.example.yml) for the full
file; the core is:

```yaml
services:
  app:
    image: example/app:1.2.3           # pinned, not :latest (entry 19: @sha256)
    user: "1000:1000"                   # don't run as root inside  (entry 02)
    read_only: true                     # rootfs is immutable       (entry 04)
    tmpfs:
      - /tmp                            # give back just the writable bits it needs
    cap_drop:
      - ALL                             # drop every Linux capability(entry 03)
    # cap_add:
    #   - CHOWN                         # add back ONLY what the app actually needs
    security_opt:
      - no-new-privileges:true          # can't gain privileges via setuid (entry 03)
    mem_limit: 512m
    pids_limit: 200                     # contain fork bombs / runaway procs (entry 05)
    networks: [frontend]

  db:
    image: postgres:16-alpine
    networks: [backend]                 # DB is not on frontend...

networks:
  frontend:
  backend:
    internal: true                      # ...and backend has NO route out (entry 06)
```

What each line buys you, in one sentence:

| Line | What it stops |
|------|---------------|
| `user: "1000:1000"` | root-in-container → root-on-host if something escapes |
| `read_only: true` + `tmpfs` | malware writing a payload to the container filesystem |
| `cap_drop: [ALL]` | the container using kernel capabilities it never needed |
| `no-new-privileges:true` | a process escalating via setuid binaries |
| `mem_limit` / `pids_limit` | one container DoS-ing the whole host |
| `backend: internal: true` | your database quietly calling out to the internet |

## How to verify it worked

```bash
# running as the right non-root user?
docker exec app id
# → uid=1000 gid=1000   (NOT uid=0 root)

# capabilities actually dropped?
docker inspect app --format '{{ .HostConfig.CapDrop }}'
# → [ALL]

# rootfs really read-only? (should FAIL)
docker exec app sh -c 'touch /test' 
# → touch: /test: Read-only file system

# the internal backend has no way out? (should FAIL / hang)
docker exec db sh -c 'wget -qO- https://example.com' ; echo "exit=$?"
# → non-zero exit, no response
```

If an app breaks under this, loosen **one** line at a time (usually: add a
specific `cap_add`, or drop `read_only` and add a named volume) — don't throw
the whole block away.

## Where this doesn't apply

Some images legitimately can't take all of it — e.g. LinuxServer.io images run
an s6 init as root inside the container, so `read_only` / `user:` fight them.
For those, lean harder on the *other* defenses: keep them off `privileged`, no
socket, behind a reverse proxy, on isolated networks. That trade-off is exactly
what the next entries are about.

## Links

- Docker security docs: https://docs.docker.com/engine/security/
- OWASP Docker Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html
- `docker run` security options: https://docs.docker.com/reference/cli/docker/container/run/#security-opt
