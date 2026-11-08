# 04 — Read-only root filesystem + tmpfs

Entries 02–03 shrank *who* the container is and *what powers* it holds. This one
shrinks what it can **change**: make the container's filesystem read-only, and
hand back only the few writable scratch areas it genuinely needs.

## The risk: a writable container is a place to land

When an attacker gets code execution inside a container, their first move is
usually to **write something to disk** — drop a cryptominer, a reverse shell, a
modified binary, a persistence script, a second-stage payload pulled from the
internet. All of that assumes the filesystem is writable. By default it is: a
container can write anywhere its user has permission.

A **read-only root filesystem** takes that away. The application's code and the
OS inside the image become immutable at runtime — nothing can be dropped,
overwritten, or tampered with. Malware that lands has nowhere to install itself
and doesn't survive a restart. It's one of the highest-value, lowest-effort
hardening moves for a stateless or near-stateless service.

The catch: almost every app needs to write *somewhere* (a temp dir, a PID file,
a cache). So you don't just flip it on — you flip it on **and** give back
exactly those paths as `tmpfs` (in-RAM, wiped on stop) or as named volumes (for
real data).

## The fix

```yaml
services:
  app:
    image: example/app:1.2.3
    user: "1000:1000"            # entries 02–03
    cap_drop: [ALL]
    security_opt:
      - no-new-privileges:true

    read_only: true              # the whole rootfs is now immutable
    tmpfs:
      - /tmp                     # give back a scratch dir (in RAM, ephemeral)
      # - /run                   # some apps want a writable /run for a pidfile
    volumes:
      - app-data:/data           # REAL state goes to a named volume, not the rootfs

volumes:
  app-data:
```

Two kinds of "writable" to keep straight:

- **`tmpfs`** → throwaway scratch (temp files, caches, lock/pid files). Lives in
  RAM, vanishes on container stop. Perfect for things you don't want to keep.
- **named volume** → data that must persist (a database, uploads, config). Only
  mount the *specific* path the app writes, not the whole rootfs.

The goal: the image's own files are frozen; only the handful of data paths are
writable, and each is explicitly chosen.

## How to find which paths need to be writable

Turn `read_only: true` on, start the container, and watch it fail — the errors
name the paths:

```bash
docker compose up        # (foreground, watch the logs)
# e.g.  "cannot create /tmp/xxx: Read-only file system"
# e.g.  "unable to write pidfile /run/app.pid"
```

Add each one as a `tmpfs` (if it's scratch) or a volume (if it's data), restart,
repeat until it comes up clean. Usually it's just `/tmp` and sometimes `/run`.

## How to verify it's actually read-only

```bash
# writing to the rootfs should FAIL:
docker exec <container> sh -c 'touch /evil' 
# -> touch: /evil: Read-only file system      ✅

# the scratch path you granted should WORK:
docker exec <container> sh -c 'touch /tmp/ok && echo writable'
# -> writable                                  ✅

# confirm the flag from the host:
docker inspect <container> --format '{{ .HostConfig.ReadonlyRootfs }}'
# -> true
```

## When it doesn't fit (and what to do instead)

- **Apps that write all over their install dir** (many PHP apps, some that
  self-update) fight read-only hard. Don't force it — you'd end up tmpfs-ing
  half the filesystem, which defeats the point.
- **LinuxServer.io images** (today's Calibre-Web, plus Grocy/Sonarr/Radarr…):
  their s6 init writes to `/run`, `/var`, `/config` as root at startup, so a
  blanket `read_only: true` breaks boot. That's why those stacks ship the
  hardening block **commented** — it's a deliberate Level-3 call (see the
  rubric's "applying" note), and you lean on reverse proxy + network isolation
  + no-socket instead.
- **Good fits:** stateless or single-data-dir apps — static sites, APIs,
  Umami, Vikunja, SearXNG, PairDrop (non-LSIO) — take `read_only` cleanly with
  just a `/tmp` tmpfs.

Match the lockdown to the image; don't cargo-cult it onto one that can't take it.

## Today's example

Calibre-Web is a LinuxServer image, so it's the *counter*-example again: its s6
init needs to write at startup, so `read_only` stays commented and we rely on
the other layers. The entry pairs with it precisely to show the boundary —
read-only is powerful, but it's an image-by-image decision, not a blanket rule.

## Links

- Docker `--read-only`: https://docs.docker.com/engine/containers/run/#read-only
- Compose `read_only` / `tmpfs`: https://docs.docker.com/reference/compose-file/services/#read_only
- tmpfs mounts: https://docs.docker.com/engine/storage/tmpfs/
