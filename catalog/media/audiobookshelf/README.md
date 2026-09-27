# Audiobookshelf Service

<p align="center">
  <img src="../../../assets/logos/audiobookshelf.svg" width="180">
</p>

<br>


This directory contains an example configuration for deploying the
Audiobookshelf service using Docker Compose.

Audiobookshelf is a self-hosted audiobook and podcast server designed
for managing and streaming audiobook collections.

Official project:
https://github.com/advplyr/audiobookshelf

## Purpose in this lab

- Manage audiobook libraries
- Stream audio content from a self-hosted server
- Demonstrate containerized media services

## Network

This stack expects an external Docker network.

Create it before deployment:

docker network create media_network

## Storage Design

The container separates application data into several mount points:

- /config → application configuration
- /metadata → generated metadata and library data
- /audiobooks → audiobook media library
- /podcasts → podcast media library

This separation allows easier backup strategies and media storage
management.

## Network

The service connects to the internal Docker network used by other
media containers:

your_network

This allows communication with reverse proxy services while keeping
media containers isolated from the host.

## Deploymen
docker compose up -dt

## Thanks

Thanks to the Audiobookshelf maintainers for creating a lightweight and
feature-rich audiobook management system suitable for self-hosted
environments.
