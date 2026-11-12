# 06 — Network segmentation: internal networks & no egress

The earlier entries locked down a container's privileges and filesystem. This
one controls its **network reach** — who it can talk to, and crucially whether
it can talk to the *internet at all*. Most of your containers have no business
calling out, and cutting that off is one of the strongest anti-exfiltration,
anti-phone-home moves you have.

## The risk: a compromised container with a clear line out

By default, every container Docker starts can reach the **internet** (outbound)
and **every other container on its network**. Think about what that hands an
attacker who lands inside one:

- **Exfiltration** — your database, your files, your secrets get POSTed to a
  server they control. The data doesn't need to escape the container; it just
  needs a route *out*.
- **Phone-home / C2** — a malicious image quietly beacons to a command server,
  pulls a second-stage payload, joins a botnet.
- **Lateral movement** — from the one container they popped, they scan and hit
  every other service sharing the network.

Here's the key insight: **most containers never need outbound internet.** A
database only talks to its app. A cache only talks to its app. A backend worker
only talks to its queue. If none of them can reach the internet, an attacker who
compromises one has nowhere to send the loot and no way to pull more tools.

## The fix: split the network, make the back one `internal`

```yaml
services:
  app:
    image: example/app:1.2.3
    networks: [frontend, backend]   # reachable by users; can also talk to the db

  db:
    image: postgres:16-alpine
    networks: [backend]             # ONLY on backend — no user access, no internet

networks:
  frontend:                         # normal network: has a route out / is proxied
  backend:
    internal: true                  # NO gateway to the outside world at all
```

What `internal: true` actually does: Docker creates the network **without a
gateway to the host's external interface**. Containers on it can talk to *each
other*, but there is **no route to the internet** and nothing outside the stack
can reach in. The database can serve the app and nothing else — it can't be
reached from your LAN, and if it's compromised it **cannot call home**.

The pattern:

- **`frontend`** — services that must be reachable (the web app) and/or must
  reach out. Your reverse proxy fronts this.
- **`backend` (`internal: true`)** — databases, caches, internal-only workers.
  No egress, no inbound from outside.
- An app that bridges both sits on **both** networks; the datastore sits on
  **backend only**.

## When a service genuinely needs egress

Some services *must* reach the internet — and that's fine, you just don't put
them on the internal network. Examples:

- **Speedtest Tracker** (today's service) — measuring your line speed *is*
  reaching out; it lives on `frontend` by necessity.
- Anything that fetches: RSS readers, *arr indexers, update checkers,
  notification senders, ACME/Let's Encrypt clients.

The goal isn't "nothing has egress" — it's "**only what needs egress has
egress.**" Databases, caches and internal workers almost never qualify.

## How to verify

```bash
# The internal service should FAIL to reach the internet:
docker compose exec db sh -c 'wget -qO- --timeout=5 https://example.com; echo "exit=$?"'
# -> hangs/times out, non-zero exit        ✅ no egress

# ...but can still reach the app over the shared backend network:
docker compose exec db sh -c 'getent hosts app || ping -c1 app'
# -> resolves/reachable                     ✅ intra-stack still works

# Confirm the network has no gateway to the outside:
docker network inspect <project>_backend --format '{{ .Internal }}'
# -> true
```

If the `wget` *succeeds*, the service has egress it probably shouldn't — check
it's actually on the `internal` network and not also on `frontend`.

## Going further (beyond the compose basics)

- **Don't publish ports you don't need.** A `ports:` mapping exposes a service
  to your whole LAN (and bypasses UFW — that's the next entry, 07). If only the
  reverse proxy needs to reach a service, don't publish its port at all; let the
  proxy reach it over a shared Docker network.
- **One stack, one network boundary.** Keep each app's services on their own
  networks rather than dumping everything on the default bridge, so a
  compromise in one stack can't see another.
- **Egress filtering** (advanced): for services that need *some* outbound but
  not the whole internet, a firewall/proxy that allowlists destinations is the
  next level — overkill for most homelabs, good to know it exists.

## Today's example

Speedtest Tracker is the honest counter-case: it's the service that **must**
keep egress, so it sits on a plain `frontend` network with no `internal` trick.
It proves the rule by being the exception — you segment what *can* be segmented
(databases, caches), and you leave egress only where the job demands it. Compare
it with Mattermost (entry 05's example), where the Postgres container *is* on an
`internal: true` backend with no route out.

## Links

- Docker internal networks: https://docs.docker.com/engine/network/drivers/bridge/#use-the-default-bridge-network
- Compose `internal`: https://docs.docker.com/reference/compose-file/networks/#internal
- Container networking overview: https://docs.docker.com/engine/network/
