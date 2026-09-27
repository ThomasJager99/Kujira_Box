# Glances


<p align="center">
  <img src="../../../assets/logos/glances-light.svg" width="180">
</p>

<br>


Glances is a lightweight, cross-platform monitoring tool that provides real-time
system and container metrics through a web-based interface.

This directory contains an example Docker Compose configuration used to deploy
Glances as part of a containerized infrastructure for observability purposes.

## Project & Image
Official Glances project:
https://github.com/nicolargo/glances

Docker image used:
- nicolargo/glances

## Deployment Notes
- Runs as a Docker container
- Accesses Docker metrics via read-only Docker socket
- Uses read-only mounts for host system visibility
- Connected to an internal Docker network
- Intended for internal monitoring, not public exposure

## Thanks
Thanks to the Glances maintainers for providing a simple and powerful monitoring
tool that fits well into container-based environments.
