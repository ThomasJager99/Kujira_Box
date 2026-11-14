# 08 — The Docker socket is root: use a socket-proxy

Loads of homelab tools ask you to mount `/var/run/docker.sock` — dashboards
(Homepage, Dozzle), auto-updaters (Watchtower), management UIs (Portainer,
Dockge), reverse proxies (Traefik). It's so common it feels routine. It is the
single most dangerous mount in all of Docker, and this entry is about why — and
how to give those tools what they need without handing over the keys.

## The risk: the socket *is* root on the host

`/var/run/docker.sock` is the Docker daemon's API. The daemon runs as **root**,
and anything that can talk to that socket can tell it to do anything — including
things that trivially become root on the host:

```bash
# a container with the socket mounted can do this:
docker run -v /:/host -it alpine chroot /host sh
#            ^ mount the HOST'S ENTIRE FILESYSTEM into a new container, as root
```

That's it. One command and the "isolated" container owns your whole machine:
read every file, rewrite `/etc/shadow`, install a rootkit, add an SSH key. So
when you mount the socket into a container:

- you've given **that container root on the host**, full stop;
- and if that container is **ever compromised** (a CVE in the dashboard, a
  malicious image update, a supply-chain attack), the attacker inherits it.

Mounting the socket read-only (`:ro`) **does not help** — the Docker API is all
one endpoint; "read-only" on the file doesn't stop API calls that create
privileged containers. Don't rely on it.

## The fix: a socket-proxy that exposes only what's needed

Most tools only need to *read* a slice of the API — list containers, read logs,
watch events. They don't need to create privileged containers. A
**socket-proxy** sits in front of the real socket and allows only the specific
API endpoints you turn on, denying everything else.

[`tecnativa/docker-socket-proxy`](https://github.com/Tecnativa/docker-socket-proxy)
is the standard one — a tiny HAProxy that whitelists API sections via env vars:

```yaml
services:
  dockerproxy:
    image: tecnativa/docker-socket-proxy:0.3.0
    container_name: dockerproxy
    environment:
      CONTAINERS: 1        # allow GET /containers (what a dashboard needs)
      IMAGES: 1
      NETWORKS: 1
      SERVICES: 1
      TASKS: 1
      # everything else defaults to 0 (denied): POST, EXEC, no container create
      POST: 0              # read-only: no state-changing calls at all
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro   # ONLY the proxy touches the real socket
    security_opt:
      - no-new-privileges:true
    networks: [socket]
    restart: unless-stopped

  dashboard:
    image: example/dashboard:1.2.3
    environment:
      # point the tool at the PROXY, not the real socket:
      DOCKER_HOST: tcp://dockerproxy:2375
    networks: [socket]
    restart: unless-stopped

networks:
  socket:
    internal: true         # the proxy ↔ tool channel needs no egress
```

Now the dashboard can list containers but **cannot** create one, exec into one,
or mount your filesystem — the proxy simply refuses those API calls. Only the
proxy container touches the real socket, and you keep *it* minimal, non-exposed,
and locked down.

### What to allow for common tools

| Tool | Typically needs | Keep off |
|---|---|---|
| Dozzle / log viewers | `CONTAINERS`, `POST=0` | everything else |
| Homepage / dashboards | `CONTAINERS`, `SERVICES`, `TASKS`, `POST=0` | `EXEC`, `POST` |
| Portainer / Dockge | more (they manage) — bigger attack surface | still deny `EXEC` where you can |

Grant the **narrowest** set that makes the tool work. Start with `POST: 0`
(read-only) and only open more if the tool genuinely needs to change state.

## The exception the proxy can't save

Some tools *must* create and run containers — that's their whole job:

- **Watchtower** (pulls + recreates containers to update them)
- **CI agents** (Woodpecker's agent, entry 02 — runs pipeline steps as
  containers)

A socket-proxy can't help these, because the dangerous capability (*create
containers*) is exactly the one they need. For those: run them on a
**disposable / dedicated host**, limit what they manage, and treat that host as
already-untrusted (this is where block C — VMs/sandboxes — comes in).

## How to verify

```bash
# through the proxy, a read call works:
docker -H tcp://localhost:2375 ps            # (if you temporarily expose it to test)
# but a create/exec is refused by the proxy:
docker -H tcp://localhost:2375 run alpine    # -> 403 Forbidden from the proxy

# confirm the app talks to the PROXY, not the socket:
docker inspect dashboard --format '{{ range .Config.Env }}{{ println . }}{{ end }}' | grep DOCKER_HOST
# -> DOCKER_HOST=tcp://dockerproxy:2375    (NOT a socket mount)

# confirm ONLY the proxy mounts the real socket:
docker inspect dashboard --format '{{ .Mounts }}'   # should show NO docker.sock
```

## Today's example

Today's service is a **private Docker registry** — notably, it does *not* need
the socket at all (it's just storage), which is the point: most things don't.
The socket-mounting tools are a specific minority, and for them the proxy is the
answer. Before you paste `- /var/run/docker.sock:/var/run/docker.sock` into a
compose file, stop and ask: does this tool need to *control* Docker, or just
*read* it? If it's "read," it belongs behind a socket-proxy.

## Links

- `docker-socket-proxy`: https://github.com/Tecnativa/docker-socket-proxy
- Why the socket = root (Docker docs): https://docs.docker.com/engine/security/#docker-daemon-attack-surface
- OWASP Docker Cheat Sheet (Rule 1): https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html
