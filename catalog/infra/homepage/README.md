<p align="center">
  <img src="../../../assets/logos/homepage.png" width="180">
</p>

<br>


This directory contains an example deployment of **Homepage**, a lightweight
dashboard application used to organize and access internal services.

Homepage provides a customizable web interface where links to services,
monitoring dashboards and tools can be grouped into sections.

## Image

Official Homepage container image:

https://github.com/gethomepage/homepage

## Deployment Notes

- Runs as a standalone dashboard container
- Stores configuration in a persistent Docker volume
- Exposes a single HTTP interface
- Can be used as a landing page for internal infrastructure tools

## Example Use

Homepage can be used to organize access to services such as:

- monitoring dashboards
- media services
- development tools
- internal web applications

This allows administrators to centralize access to commonly used services
through a single interface.

## Thanks

Thanks to the Homepage project maintainers for providing a clean and simple
dashboard solution suitable for self-hosted environments.
