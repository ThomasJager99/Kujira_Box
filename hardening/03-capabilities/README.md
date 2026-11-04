# 03 — Capabilities: `cap_drop: [ALL]` + `no-new-privileges`

Entry 02 stopped the container from *being* root. This entry handles the next
question: even a non-root process can be granted slices of root's power, called
**Linux capabilities**. Here we take them away — and slam the door on getting
them back.

## The risk: "root" is really ~40 separate superpowers

Classic Unix had a binary switch: you were root (could do everything) or not
(could do almost nothing). Linux broke root's power into ~40 discrete
**capabilities** you can hand out one at a time, e.g.:

- `CAP_NET_RAW` — craft raw packets (ping, but also ARP spoofing / sniffing)
- `CAP_NET_BIND_SERVICE` — bind ports below 1024
- `CAP_SYS_ADMIN` — the "almost root again" grab-bag (mount, namespaces…)
- `CAP_DAC_OVERRIDE` — bypass file permission checks
- `CAP_SETUID` / `CAP_SETGID` — become another user

Docker is reasonably sane here: by default it **drops most** capabilities and
leaves a small default set. But that default set is still more than most apps
need — and `CAP_NET_RAW` and `CAP_SETUID` in that default set are exactly the
kind of thing malware reaches for. A web app that serves pages on port 8080 has
no business holding *any* of them.

Two independent moves close this:

1. **Drop every capability, then add back only what's needed.**
2. **`no-new-privileges`** — forbid the process from ever *gaining* new
   privileges later (e.g. via a setuid binary baked into the image).

## The fix

```yaml
services:
  app:
    image: example/app:1.2.3
    user: "1000:1000"              # from entry 02
    cap_drop:
      - ALL                        # start from zero — hold NO capabilities
    # cap_add:                     # ...then grant back only the few it proves it needs
    #   - NET_BIND_SERVICE         # e.g. only if it must bind :80 / :443
    security_opt:
      - no-new-privileges:true     # can never escalate, even via setuid
```

`cap_drop: [ALL]` is safe to try on almost anything first — most apps that run
unprivileged on a high port need **nothing** added back. If one breaks, the
error tells you which capability to `cap_add` (add them one at a time, never
`SYS_ADMIN` unless you truly understand why).

### `no-new-privileges` — why it's separate

Dropping capabilities controls what the process *starts* with.
`no-new-privileges` controls whether it can *acquire* more afterwards. Without
it, a setuid-root binary inside the image (like `sudo`, `ping`, `mount`) can
bump a process back up. With it set, the kernel refuses — the
`execve()`-gains-privileges path is dead. It costs nothing and should be on
**every** container (it's in the baseline block from entry 01 for that reason).

## How to verify

```bash
# What capabilities does the running container actually hold?
docker inspect <container> --format '{{ .HostConfig.CapDrop }} / {{ .HostConfig.CapAdd }}'
# want:  [ALL] / []     (dropped all, added nothing)

# Confirm from inside — install libcap or read the proc bits:
docker exec <container> grep CapEff /proc/1/status
# CapEff: 0000000000000000   ← all-zero = no effective capabilities

# no-new-privileges applied?
docker inspect <container> --format '{{ .HostConfig.SecurityOpt }}'
# want: [no-new-privileges:true]
```

`CapEff: 0000000000000000` is the gold star — the main process holds zero
capabilities.

## Common add-backs (and the ones to fear)

| App needs to… | Add back |
|---|---|
| bind port < 1024 (serve :80 directly) | `NET_BIND_SERVICE` (or just use a high port) |
| run `ping` / ICMP | `NET_RAW` |
| change file owner at startup | `CHOWN`, maybe `FOWNER` |
| drop to another user (some entrypoints) | `SETUID`, `SETGID` |

🚩 **Red flags:** `SYS_ADMIN`, `SYS_PTRACE`, `NET_ADMIN`, or `privileged: true`.
If an image *demands* `SYS_ADMIN` or `privileged`, that's a near-root container
— treat it like the Woodpecker agent (entry 02): isolate the host, don't run it
next to anything you care about.

## Today's example

PairDrop (today's service) is a perfect candidate: it's a stateless web app on
a high port, so `cap_drop: [ALL]` with nothing added back is realistic. (Its
LinuxServer packaging runs an s6 init as root, which complicates a full lockdown
— but the *principle* is exactly this, and a non-LSIO image of it would take
`cap_drop: [ALL]` cleanly.)

## Links

- Linux capabilities man page: https://man7.org/linux/man-pages/man7/capabilities.7.html
- Docker `--cap-drop` / `--cap-add`: https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities
- `no-new-privileges`: https://docs.docker.com/reference/cli/docker/container/run/#security-opt
