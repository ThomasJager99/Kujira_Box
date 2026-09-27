This directory contains an example deployment of Immich, a high-performance, open-source, self-hosted photo and video backup solution — a private alternative to Google Photos. Immich backs up the photos and videos from your phone automatically (with polished iOS and Android apps), then gives you a fast timeline, albums, a map view, and — thanks to on-device-style machine learning running on your own server — face recognition and smart search ("show me photos of the beach"). Everything, including the AI features, runs entirely on hardware you control, so your family photos never leave your server.


<p align="center">
  <img src="../../../assets/logos/immich.svg" width="180">
</p>

<br>


## Official container image

https://github.com/immich-app/immich — images published under `ghcr.io/immich-app/*`.

## Deployment Notes

- Runs as a four-container stack, each with one job:
  - **immich-server** — the API, web UI and background workers.
  - **immich-machine-learning** — runs the ML models for face recognition and smart/semantic search (kept separate because the models are heavy).
  - **redis** — a Valkey (Redis-compatible) cache/queue for background jobs.
  - **database** — a special PostgreSQL image with vector extensions (`vectorchord` / `pgvecto.rs`) that powers similarity search.
- Photos/videos and the database live on **host bind mounts** (`UPLOAD_LOCATION`, `DB_DATA_LOCATION`) — not named volumes — so you know exactly where your originals sit. The ML model cache uses a named volume (`model_cache`).
- **The database path must be a real local filesystem** — network shares (NFS/SMB) are unsupported and corrupt the vector database.
- Startup ordering is enforced with healthchecks: the server only starts once the database and redis report healthy. The server and ML images ship their own built-in healthchecks; redis and the database use explicit ones (`redis-cli ping`, `pg_isready`).
- Exposes a single HTTP interface (container listens on port 2283); intended to sit behind a reverse proxy for TLS in production.
- Memory and CPU limits are set on every service (ML gets the most — the models are the hungry part).
- Environment-specific values (version, pinned image digests, port, storage paths, database credentials) are externalized to a `.env` file. **Pin `IMMICH_VERSION`** to a specific release rather than tracking `release`, so upgrades are deliberate.
- **Backups:** back up the `UPLOAD_LOCATION` folder (your actual media) and a `pg_dump` of the database. The `model_cache` is disposable — it re-downloads.

## Example Use

Immich can be used to:

- automatically back up photos & videos from phones (iOS / Android apps)
- browse a fast timeline, albums, favourites and a photo map
- search by content and recognize faces/people with self-hosted ML
- share albums with family or via public links
- free yourself from cloud photo subscriptions while keeping the same convenience

This gives a household a private, feature-rich photo library that replaces Google Photos or iCloud Photos, on its own hardware.

## Thanks

Thanks to the Immich project and its very active community for building a genuinely polished, self-hosted photo platform — mobile apps, machine learning and all — as open source.

## Links

- Source: https://github.com/immich-app/immich
- Website: https://immich.app
- Documentation: https://docs.immich.app
