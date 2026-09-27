# nginx

<p align="center">
  <img src="../../../assets/logos/nginx.svg" width="180">
</p>

<br>

nginx is used as a reverse proxy and the single ingress point for internal
services running inside a Docker-based infrastructure.

This directory contains an example configuration demonstrating path-based
routing to expose internal services without publishing their ports directly.

## Image
Official nginx image:
https://hub.docker.com/_/nginx

## Deployment Notes
- Runs as a dedicated reverse proxy container
- Only exposed service to the host
- Routes traffic to internal services via Docker DNS
- Uses path-based routing instead of multiple exposed ports

## Example
The Glances monitoring interface is exposed at:

http://localhost/glances/

while the Glances container itself remains internal-only.

## Thanks
Thanks to the nginx maintainers for providing a reliable and flexible reverse
proxy that fits well into containerized environments.
