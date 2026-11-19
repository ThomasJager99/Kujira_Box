# 12 — Rootless Docker

Everything so far hardened individual containers. This entry hardens the thing
*underneath* them: the Docker daemon itself. By default it runs as **root**, and
that root is the soft underbelly of the whole setup. Rootless Docker runs the
daemon — and your containers — as an ordinary user, so a container escape lands
in an unprivileged account instead of on root.

## The risk: the daemon is root, so an escape is game over

Normally `dockerd` runs as root on the host. That means:

- Every container is, at the kernel level, running under a **root-owned
  daemon**. A container escape (a kernel CVE, a misconfig, a mounted socket —
  entry 08) puts the attacker at **root on the host**.
- Anyone in the `docker` group effectively **has root** without a password —
  `docker run -v /:/host ...` and they own the machine (entry 08). The group is
  root-equivalent.

All the per-container hardening (entries 02–06) shrinks what a *compromised
container* can do — but if the daemon under it is root, the worst case is still
"attacker becomes host root." Rootless Docker changes the worst case.

## What rootless Docker does

Rootless mode runs `dockerd` and your containers **as a normal, unprivileged
user**, using Linux **user namespaces** to map container-UID-0 to an
*unprivileged* host UID. So:

- "root inside the container" = some high, **unprivileged** UID on the host (e.g.
  165536), owning nothing it shouldn't.
- A container escape lands as **that unprivileged user**, not host root. The
  blast radius drops from "whole machine" to "one user's stuff."
- A compromise of the daemon itself is no longer an automatic host takeover.

It's the single biggest reduction in worst-case impact you can make to a Docker
host — it changes the *ceiling* on how bad a breakout can be.

## The trade-offs (why it isn't the default yet)

Rootless isn't free; know the rough edges before switching:

- **Privileged ports (<1024).** A rootless daemon can't bind `:80`/`:443`
  directly without extra setup (`setcap` on rootlesskit, or a sysctl). In
  practice you publish high ports and let a reverse proxy (also rootless, or on
  the host) front them — which you're doing anyway (entry 09).
- **Networking/storage nuances.** Uses slirp4netns / overlayfs-in-userns;
  slightly slower networking, and a few features (some volume plugins, certain
  `--net=host` tricks) differ or need config.
- **Not everything likes it.** A handful of images that genuinely need host
  privileges won't run rootless — those were already your "disposable host"
  cases (entry 08's Watchtower, CI agents).

For a typical web-app homelab behind a proxy, none of these are blockers.

## How to enable it (outline)

```bash
# as your normal user (NOT root), on the host:
curl -fsSL https://get.docker.com/rootless | sh
# it sets up a per-user dockerd; follow the printed env lines, e.g.:
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock

# start it on login + keep it running after logout:
systemctl --user enable --now docker
sudo loginctl enable-linger $(whoami)

# (optional) allow binding low ports:
#   sudo setcap cap_net_bind_service=ep $(which rootlesskit)
```

Now `docker` talks to *your* user's daemon. Your compose stacks run unchanged;
they're just no longer backed by root.

## Podman: rootless by design (next entry)

If you're setting this up fresh, **Podman** (entry 13) is rootless and
daemonless *by default* and is largely Docker-CLI-compatible — worth knowing
before you invest in retrofitting Docker. This entry is for keeping Docker and
removing its root; the next weighs Podman as the alternative.

## How to verify

```bash
# the daemon is running as YOU, not root:
docker info -f '{{.SecurityOptions}}'
# -> includes  name=rootless     ✅

ps -o user= -p "$(pgrep -n dockerd)"
# -> your username, NOT root

# container-root maps to an unprivileged HOST uid:
docker run --rm alpine sh -c 'id -u'     # 0 inside...
docker top <container>                    # ...but the host process runs as your mapped uid, not 0
```

## Today's example

Ghost (today's service) runs fine under rootless Docker — it's a normal web app
on a high port (2368) behind your reverse proxy, which is exactly the shape
rootless handles without fuss. The MySQL beside it, the content volume, the
internal network — all unchanged. You get the same stack with the worst-case
outcome of a breakout downgraded from "root on the box" to "one unprivileged
user." That's the whole pitch.

## Links

- Rootless mode (Docker docs): https://docs.docker.com/engine/security/rootless/
- User namespaces: https://docs.docker.com/engine/security/userns-remap/
- rootlesskit: https://github.com/rootless-containers/rootlesskit
