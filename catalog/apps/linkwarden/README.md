This directory contains an example deployment of Linkwarden, a self-hosted
bookmark manager that doesn't just save links — it **archives** them. When you
add a bookmark, Linkwarden captures a full copy of the page (a screenshot, a PDF
and the readable text), so the content survives even if the original site goes
down or changes. You organize everything into collections and tags, search across
the full text of every saved page, and reach it all from a clean web UI, a
browser extension and mobile apps — a private alternative to cloud "read it
later" and bookmarking services, on hardware you control.


<p align="center">
  <img src="../../../assets/logos/linkwarden.png" width="180">
</p>

<br>


## Official container image

https://github.com/linkwarden/linkwarden — image published as `ghcr.io/linkwarden/linkwarden`.

## Deployment Notes

- Runs as a three-container stack:
  - **linkwarden** — the app (web UI, archiving engine).
  - **postgres** — stores bookmarks, collections, tags and metadata.
  - **meilisearch** — powers fast full-text search across the archived page content.
- Stores state in persistent volumes: the app's uploads and page archives
  (`/data/data`), the PostgreSQL database, and the Meilisearch index.
- Startup ordering is enforced with healthchecks: the app waits for PostgreSQL to
  report healthy (and for Meilisearch to start) before booting.
- Exposes a single HTTP interface on port `3000`; put a reverse proxy in front for
  TLS when reached over the internet.
- `NEXTAUTH_SECRET` (session encryption) and `MEILI_MASTER_KEY` must be set to
  strong random values; `NEXTAUTH_URL` must match the address you actually use.
- Meilisearch analytics are disabled (`MEILI_NO_ANALYTICS`).
- Memory and CPU limits are set on every service.
- Environment-specific values (images, port, URL, secrets, DB password) are in `.env`.
- **Backups:** a `pg_dump` of the database plus the app's `/data/data` volume (the
  archived pages and uploads) together are a complete restore.

## Example Use

Linkwarden can be used to:

- save bookmarks and automatically archive each page (screenshot + PDF + text)
- keep links readable forever, even after the original site dies or changes
- organize links into collections with tags and descriptions
- search the full text of everything you've saved, not just titles
- capture pages from a browser extension or the mobile apps
- share collections publicly or with collaborators
- replace cloud bookmarking / read-it-later services with a private one

This gives an individual or team a durable, searchable, self-hosted knowledge
library of everything worth keeping from the web.

## Thanks

Thanks to the Linkwarden project and its maintainers for a polished, genuinely
useful bookmark-and-archive tool built as open source.

## Links

- Source: https://github.com/linkwarden/linkwarden
- Website: https://linkwarden.app
- Documentation: https://docs.linkwarden.app
