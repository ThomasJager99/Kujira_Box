# Dozzle

**Dozzle** is a tiny, no-fuss web UI for watching your Docker container logs in
real time. Point it at the Docker socket and it instantly shows every running
container with a live, colourised, searchable log stream — no database, no log
shipping, no agents to install. It's not a long-term log store (that's a job for
Loki/Grafana); it's the tool you open when you want to *see what a container is
doing right now* — tailing output, following a restart loop, grepping an error —
all from the browser instead of SSH-ing in and typing `docker logs -f`. For a
homelab it's a near-zero-cost quality-of-life win: one small container and every
other container's logs are a click away.


<p align="center">
  <img src="../../../assets/logos/dozzle.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `amir20/dozzle`
- **Project:** https://github.com/amir20/dozzle
- **Docs:** https://dozzle.dev/

## Deployment Notes

- **One container.** Single small `dozzle` service — stateless, no database, no
  persistent volumes.
- **Docker socket, read-only.** Mounts `/var/run/docker.sock` as **`:ro`** —
  Dozzle only needs to *read* logs, never to control Docker. Note that even
  read access to the socket exposes information about every container; keep the
  UI off the public internet. For a hardened setup, put a
  **docker-socket-proxy** in front and expose only the containers/logs
  endpoints.
- **Single HTTP interface** on port `8080`.
- **Auth.** Disabled by default (`DOZZLE_AUTH_PROVIDER=none`) — fine on a trusted
  LAN or behind a reverse proxy that already authenticates. Dozzle also ships a
  built-in `simple` auth provider (username/password via a users file) if you
  want login without a proxy.
- **Healthcheck.** The image is distroless (no shell or wget), so the check runs
  the binary's own `dozzle healthcheck` subcommand.
- **Resource limits** (`mem_limit` / `mem_reservation` / `cpus`) and all
  environment-specific values are externalized to `.env`.

## Example Use

- Tail a service's logs live while you reproduce a bug — no SSH needed.
- Watch a container that's stuck in a restart loop and read why it's crashing.
- Search across a container's recent output for an error string.
- Give yourself (or a teammate) a read-only window into logs without handing out
  shell access to the host.

## Thanks

To **Amir Raminfar** and the Dozzle contributors for a fast, dependency-free way
to read container logs.

## Links

- Project: https://github.com/amir20/dozzle
- Documentation: https://dozzle.dev/
- Security / auth: https://dozzle.dev/guide/authentication
