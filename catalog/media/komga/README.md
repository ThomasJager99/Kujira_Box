# Komga

<p align="center">
  <img src="../../../assets/logos/komga.svg" width="180">
</p>

<br>


Komga is a self-hosted media server designed for managing and reading comics,
manga, and graphic novels through a modern web interface.

This directory contains an example Docker Compose configuration used to run
Komga as part of a containerized media stack.

## Project & Image
Official Komga project:
https://github.com/gotson/komga

Docker image used:
- gotson/komga

## Deployment Notes
- Runs as a Docker container
- Uses bind mounts for persistent configuration and metadata
- Media library is mounted read-only
- Connected to an internal Docker network
- Intended to be accessed via a reverse proxy

## Thanks
Big thanks to the Komga developers for creating a clean and well-designed
self-hosted comics server, and for maintaining a solid Docker image that makes
deployment simple and reliable.
