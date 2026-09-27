<p align="center">
  <img src="../../../assets/logos/trilium.svg" width="180">
</p>

<br>

# Trilium Notes (Docker)

This directory contains a Docker Compose setup for running **Trilium Notes** as a self-hosted knowledge base.

---

## Image

Official image used:

https://hub.docker.com/r/zadam/trilium

---

## What is Trilium

Trilium Notes is a hierarchical note-taking application designed for building personal knowledge bases.

It supports:

- Tree-structured notes
- Rich text editing
- Code notes
- Internal linking between notes
- Local-first data storage

---

## Deployment Notes

- Runs as a single container service
- Uses a **named volume** for persistent storage
- Accessible via configured host port
- Designed for local or homelab usage

---

## Storage

All application data is stored inside the container at:

/home/node/trilium-data

Mapped to Docker volume:

trilium_data

---

## Access

After deployment, Trilium is available at:

http://localhost:<TRILIUM_PORT>

Example:

http://localhost:8085

---

## Configuration

Environment variables are defined in:

.env.example

Key parameters:

- `TRILIUM_PORT` – exposed port
- `TRILIUM_IMAGE` – container image
- `TRILIUM_MEM` – memory limit
- `TRILIUM_CPU` – CPU limit

---

## Notes

- No external database required
- Data is stored locally inside Docker volume
- Suitable for personal usage and lightweight setups

---

## Thanks

Thanks to the Trilium developers for providing a powerful and flexible knowledge management tool.

