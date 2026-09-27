This directory contains an example deployment of Portainer CE, a web-based
management UI for Docker. Portainer gives you a full graphical control panel for
your Docker host: browse and manage containers, images, volumes and networks,
read live logs, open a shell inside a running container, deploy and edit stacks,
and manage users and access — all from the browser instead of the command line.
It's a popular, recognizable tool for running a Docker host without living in the
terminal, and the Community Edition is free and open source.


<p align="center">
  <img src="../../../assets/logos/portainer-dark.svg" width="180">
</p>

<br>


## Official container image

https://github.com/portainer/portainer — image published as `portainer/portainer-ce`.

## Deployment Notes

- Runs as a single container.
- **Mounts the Docker socket (`/var/run/docker.sock`) with read-write access** so
  it can manage the daemon. This grants container-level, effectively host-root
  power — run Portainer only on trusted hosts and keep it off the open internet
  without a protecting layer (VPN / reverse proxy + auth).
- Stores its own configuration and database in the `portainer_data` volume.
- Serves an **HTTPS** web UI on port `9443` with a **self-signed certificate**, so
  the browser will show a warning on first visit (expected). Put a reverse proxy
  in front for a real certificate if desired.
- **No healthcheck is defined:** the Portainer image is minimal (no shell or HTTP
  client to run an in-container check) and the UI is HTTPS/self-signed, so we rely
  on `restart: unless-stopped` rather than a fake check.
- **First launch:** open the UI promptly and set the admin password — for security
  Portainer locks initial setup if the container sits unconfigured too long (just
  restart it if that happens).
- Memory and CPU limits keep it light.
- Environment-specific values (image tag, port) are externalized to `.env`.
- **Backups:** the `portainer_data` volume holds all settings, users and endpoints.

## Example Use

Portainer can be used to:

- manage containers, images, volumes and networks from a web dashboard
- view real-time container logs and open an interactive shell (exec) into one
- deploy and edit Docker Compose stacks through the UI
- manage multiple Docker environments/endpoints from one place
- control access with users, teams and role-based permissions
- run a Docker host day-to-day without the command line

This gives a homelab a friendly, powerful control panel for everything Docker.

## Thanks

Thanks to the Portainer team for a polished, widely-used management UI that makes
running Docker approachable, with a genuinely capable free Community Edition.

## Links

- Source: https://github.com/portainer/portainer
- Website: https://www.portainer.io
- Documentation: https://docs.portainer.io
