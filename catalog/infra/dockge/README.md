This directory contains an example deployment of Dockge, an open-source, self-hosted manager for Docker Compose stacks. Dockge gives you a clean web interface to create, edit, start, stop and monitor your `docker-compose.yml` stacks without touching the command line: you paste or write a compose file in the browser, hit deploy, and watch the container logs stream live. It keeps each stack as a plain compose file on disk (nothing is hidden in a database), so what you manage in the UI stays fully compatible with running `docker compose` by hand — Dockge is a convenience layer on top, not a lock-in.


<p align="center">
  <img src="../../../assets/logos/dockge.svg" width="180">
</p>

<br>


## Official container image

https://github.com/louislam/dockge — image published as `louislam/dockge`.

## Deployment Notes

- Runs as a single container — the web UI and the manager backend are self-contained.
- Mounts the host Docker socket (`/var/run/docker.sock`) so it can create and control containers on your behalf. **This effectively grants host-root-level power, so run Dockge only on trusted hosts and keep it off the open internet.**
- Manages your stacks as real compose files in a stacks directory (`DOCKGE_STACKS_DIR`, default `/opt/stacks`), bind-mounted at the *same path inside and outside* the container so the paths line up when it runs `docker compose`.
- Stores its own app data (settings, etc.) in a persistent Docker volume (`dockge_data`).
- Exposes a single HTTP interface (container listens on port 5001).
- Uses a Node-based healthcheck (the image ships no curl/wget, but Node is always present).
- Memory and CPU limits are set to keep it light on a shared host.
- Environment-specific values (image tag, host port, stacks directory) are externalized to a `.env` file.

## Example Use

Dockge can be used to:

- deploy and manage multiple compose stacks from one web dashboard
- edit compose files in the browser with a live editor and instantly redeploy
- start, stop, restart and remove stacks with a click
- watch real-time container logs and open an interactive terminal
- convert a `docker run ...` command into a compose file automatically
- keep an at-a-glance overview of which stacks and containers are running

This gives a homelab a friendly control panel for its Docker Compose stacks while keeping everything as portable, plain compose files.

## Thanks

Thanks to Louis Lam and the Dockge project for a lightweight, elegant stack manager that makes running Docker Compose on a homelab genuinely pleasant — while staying honest to plain compose files.

## Links

- Source: https://github.com/louislam/dockge
- Website: https://dockge.kuma.pet
