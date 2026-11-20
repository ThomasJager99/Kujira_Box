# 13 — Podman: rootless & daemonless by default

Entry 12 removed root from *Docker*. This entry looks at the alternative that
was built without it in the first place: **Podman**. If you're choosing a
container engine fresh — or hardening a host from scratch — it's worth knowing
what Podman changes and what it costs, because much of the previous entry's
effort is simply its default.

## The idea: no daemon, no root

Docker has a central **daemon** (`dockerd`) running as **root**, and every
`docker` command talks to it. That daemon is a single, privileged, always-on
process — a juicy target and a single point of failure (entries 08, 12).

Podman is **daemonless** and **rootless by default**:

- **Daemonless** — there's no long-running privileged service. `podman run`
  forks the container as a **child of your shell/session**; when it exits,
  nothing root-owned was ever sitting there. Smaller attack surface, nothing to
  compromise between commands.
- **Rootless by default** — containers run under **your** user via user
  namespaces, out of the box, no setup. "root in the container" is already a
  mapped, unprivileged host UID (the whole point of entry 12 — here it's the
  default).
- **No root-equivalent `docker` group.** There's no group that silently grants
  host root (entry 08's other danger) because there's no privileged socket to
  grant access to.

Security-wise, Podman *starts* where hardened Docker *ends up*.

## The catch: daemonless means no daemon to keep things running

The flip side of "no daemon" is "nothing is watching your containers." With
Docker, `dockerd` restarts containers (`restart: unless-stopped`) after a reboot
or crash. Podman has no daemon to do that — so on its own, a reboot leaves your
containers stopped.

The fix is to let **systemd** be the supervisor (which, on a Linux host, is
arguably cleaner anyway):

- **Quadlet** (modern, recommended) — drop a `.container` file in
  `~/.config/containers/systemd/` and systemd generates a service that starts it
  on boot, restarts on failure, and logs to the journal.
- `podman generate systemd` (older method) also works.

So "restart policies" become "systemd units." More Linux-native, a bit more to
learn.

## Compatibility: mostly a drop-in, with sharp edges

- **CLI:** `podman` mirrors the `docker` CLI closely — `alias docker=podman`
  works for a huge fraction of commands.
- **Compose:** `podman compose` (via `podman-compose` or the Docker-compatible
  API socket) runs most compose files. Complex stacks sometimes need tweaks.
- **Pods:** Podman can group containers into a **pod** (shared network
  namespace) — the concept Kubernetes uses; `podman kube` can even
  generate/consume Kubernetes YAML.
- **Rough edges:** some tools that expect a Docker *socket* (anything in entry
  08 — Portainer, Traefik's Docker provider, Watchtower) need the Podman
  compatibility socket enabled, and a few images that assume a root daemon
  behave differently. Test, don't assume.

## When to pick which

- **Keep Docker (rootless, entry 12)** if you have a working Docker setup and
  tooling that expects it — you can get most of Podman's safety without
  migrating.
- **Choose Podman** for a fresh host, when you want rootless+daemonless as the
  *default* with no ongoing effort, or on Fedora/RHEL where it's the native
  first-class engine.
- Either way, the security goal is the same as entry 12: **no root daemon under
  your containers.**

## How to verify (if you try it)

```bash
# daemonless: there is NO dockerd/podman daemon sitting around
pgrep -a dockerd        # (nothing)
podman info --format '{{.Host.Security.Rootless}}'
# -> true                ✅ rootless

# container-root maps to an unprivileged host uid, like rootless Docker:
podman run --rm alpine id -u      # 0 inside
podman unshare cat /proc/self/uid_map   # shows the 0 -> highUID mapping

# systemd (Quadlet) is supervising, not a daemon:
systemctl --user status <your-container>.service
```

## Today's example

Metabase (today's service) is a plain web-app-plus-database stack — it runs
under Podman essentially unchanged: `podman compose up -d`, or better, a Quadlet
`.container` unit each for Metabase and Postgres so systemd brings them up on
boot. The frontend/backend networks, the limits, the internal DB — all the same
compose concepts carry over. You'd just be running them with no root daemon
underneath from the start.

## Links

- Podman: https://podman.io/ · https://docs.podman.io/
- Rootless Podman: https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md
- Quadlet: https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html
