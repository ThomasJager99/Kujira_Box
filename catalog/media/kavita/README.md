# Kavita

<p align="center">
  <img src="../../../assets/logos/kavita.svg" width="180">
</p>

<br>


Kavita is a self-hosted digital library server for managing and reading e-books,
comics, and manga through a web-based interface.

This directory contains an example Docker Compose setup used to run Kavita as part
of a personal containerized infrastructure.

## Project & Image
Official Kavita project:
https://github.com/Kareadita/Kavita

Docker image used:
- jvmilazz0/kavita

## Deployment Notes
- Runs as a Docker container
- Uses bind mounts for persistent configuration and data
- Media libraries are mounted read-only
- Intended to be accessed through a reverse proxy
- No application state is stored inside the container filesystem

## Thanks
Huge thanks to the Kavita developers and maintainers for building and maintaining
such a solid self-hosted library system, and to the Docker image maintainers for
making containerized deployment straightforward.
