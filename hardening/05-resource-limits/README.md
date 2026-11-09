# 05 — Resource & PID limits

The previous entries stopped a container from gaining power or writing to disk.
This one stops a container from **starving the rest of your machine** — whether
from a bug, a runaway process, or a deliberate denial-of-service. It's the
"availability" leg of security, and it's one line each.

## The risk: one container can take down the whole host

By default a container can use **all** the host's memory, **all** its CPU, and
spawn **as many processes** as the kernel allows. So any single container —
compromised *or just buggy* — can:

- **eat all the RAM** → the kernel's OOM killer starts shooting processes, often
  the wrong ones; your other services (and sometimes the host) fall over;
- **peg every CPU core** → everything else grinds to a crawl;
- **fork bomb** → spawn processes until the process table is full and nothing
  new can start, host included.

None of these need an attacker — a memory leak or an infinite loop does it. But
they're *also* a classic post-compromise move (spin up a miner, pin the CPU)
and a cheap DoS. Limits turn "the whole box is down" into "one container got
throttled or restarted."

## The fix

```yaml
services:
  app:
    image: example/app:1.2.3
    mem_limit: 512m           # hard ceiling — container is OOM-killed if it exceeds
    mem_reservation: 128m     # soft floor — a hint for scheduling, not a cap
    cpus: 1.0                 # at most 1 full CPU core's worth of time
    pids_limit: 200           # at most 200 processes/threads — stops fork bombs
    restart: unless-stopped   # so a killed container comes back on its own
```

What each one does:

| Option | Meaning |
|---|---|
| `mem_limit` | **Hard** cap. Cross it and the container is OOM-killed (and restarted). Protects the host's RAM. |
| `mem_reservation` | **Soft** floor — a scheduling hint, *not* a limit. Keep it below `mem_limit`. |
| `cpus` | Fractional core cap. `1.5` = one and a half cores' worth of time. |
| `pids_limit` | Max processes/threads in the container. The anti-fork-bomb. |
| `restart: unless-stopped` | Pairs with the caps — if a container *is* killed for breaching one, it comes back. |

### How to size them

Don't guess once and forget — **measure**:

```bash
docker stats           # live CPU% / MEM USAGE / PIDS per container
```

Let the service run under normal load, read its real memory/PID use, then set
`mem_limit` to roughly **1.5–2×** the steady-state peak (headroom for spikes),
and `pids_limit` comfortably above its normal thread count. Too tight and you'll
get surprise OOM-kills; too loose and the limit does nothing. For databases and
JVM/Go apps especially, check the docs — some need to be *told* the limit too
(e.g. JVM heap flags), or they'll size themselves to the host and then get
killed.

## Compose v2 note

In modern Compose you can also write these under a `deploy.resources` block
(the Swarm-style syntax):

```yaml
    deploy:
      resources:
        limits: { memory: 512m, cpus: "1.0", pids: 200 }
        reservations: { memory: 128m }
```

On a single host with `docker compose up`, the **top-level short form**
(`mem_limit`, `cpus`, `pids_limit`) is the reliable one and what this rubric
uses — `deploy.limits` is honored by `docker stack deploy` (Swarm) but has been
inconsistently applied by plain `docker compose` across versions. Stick with
the short form unless you're on Swarm.

## How to verify

```bash
# the limits the container is actually running with:
docker inspect <container> --format \
  'mem={{.HostConfig.Memory}} cpus={{.HostConfig.NanoCpus}} pids={{.HostConfig.PidsLimit}}'
# mem is in bytes (512m = 536870912), pids should match your value

# watch real usage vs the limit:
docker stats <container>
# MEM USAGE / LIMIT column should show your ceiling

# prove pids_limit works (should fail to spawn past the cap):
docker exec <container> sh -c ':(){ :|:& };:' 2>&1 | head -1   # ⚠️ test box only
# fork bomb hits the pids ceiling and is contained, not the host
```

## Why it counts as security, not just ops

Availability is one of the three classic security properties (confidentiality,
integrity, **availability**). An attacker who can't steal your data may settle
for taking your services down, or quietly renting your CPU to mine coins.
Resource limits are the cheap, boring control that defuses both — and they
protect you from your *own* bugs on every other day.

## Today's example

Mattermost (today's service) ships with both `mem_limit`/`cpus` **and**
`pids_limit` on the app *and* the Postgres container — so a plugin gone wild or
a query storm on the DB can't drag the host down with it. It's the first
multi-container stack in the series where you can see the limits sized
differently per service (the app gets more RAM than the DB).

## Links

- Docker resource constraints: https://docs.docker.com/engine/containers/resource_constraints/
- Compose `mem_limit` / `cpus` / `pids_limit`: https://docs.docker.com/reference/compose-file/services/
- OOM & the kernel killer: https://docs.docker.com/engine/containers/run/#user-memory-constraints
