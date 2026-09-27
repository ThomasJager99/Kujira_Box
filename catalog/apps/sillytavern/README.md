This directory contains an example configuration for deploying SillyTavern as a containerized service within a Docker network.

<p align="center">
  <img src="../../../assets/logos/sillytavern.png" width="180">
</p>

<br>

## Image

Official container image:

https://ghcr.io/sillytavern/sillytavern

## Deployment Notes

- Runs as a standalone service inside the Docker network
- Configuration and data directories are mounted as volumes
- Container runs under a non-root user
- Healthcheck verifies that the web interface is responding
- Logging policy is applied using the shared logging anchor

## Networking

The service is attached to an external Docker network.  
This allows other containers (reverse proxies, dashboards, or monitoring tools) to access the service internally without exposing additional ports.

## Access

The SillyTavern web interface is available at:

http://localhost:8000

The port can be changed through the environment configuration.

## Data Persistence

The following directories are mounted as volumes to persist configuration and user data:

- config
- data
- backups
- plugins
- extensions

This allows the container to be recreated without losing application state.

## Thanks

Thanks to the SillyTavern maintainers for providing a powerful and extensible AI chat interface that works well in containerized environments.
