This directory contains an example deployment of Paperless-ngx, an open-source document management system that turns a pile of scanned papers and PDFs into a searchable, tagged digital archive. You drop documents into a watched folder (or scan/email them in), and Paperless-ngx runs OCR to make them full-text searchable, then automatically tags, assigns correspondents and document types, and files everything away. From the web interface you can search across every document by its content, browse by tag, and pull up any paper in seconds — all on your own server, so sensitive documents (contracts, invoices, IDs) never touch a cloud.


<p align="center">
  <img src="../../../assets/logos/paperless-ngx.svg" width="180">
</p>

<br>

## Official container image

https://github.com/paperless-ngx/paperless-ngx — image published as `ghcr.io/paperless-ngx/paperless-ngx`.

## Deployment Notes

- Runs as a five-container stack, each with one job:
  - **webserver** — the Paperless-ngx app (web UI, OCR pipeline, task workers).
  - **db** — PostgreSQL, holds all document metadata, tags and search index config.
  - **broker** — Redis, the task queue that drives background OCR/consume jobs.
  - **gotenberg** — converts office documents (Word, Excel, etc.) to PDF.
  - **tika** — extracts text and metadata from office documents.
- Stores state in persistent Docker volumes: `pgdata` (database), `media` (the archived originals + OCR'd PDFs), `data` (index/config), plus `consume` (drop-folder) and `export`.
- Startup ordering is enforced with healthchecks: the webserver only boots once `db` and `broker` report healthy, so the first-run migrations never race an unready database. `gotenberg` and `tika` are auxiliary (their images ship no HTTP client), so the webserver waits on them with `service_started`.
- Exposes a single HTTP interface (container listens on port 8000); intended to sit behind a reverse proxy for TLS in production.
- Memory and CPU limits are set on every service to keep the stack polite on a shared homelab host.
- Environment-specific values (image tags, host port, URL, OCR languages, secret key, admin credentials, database credentials) are externalized to a `.env` file.
- **Backups:** dump the PostgreSQL database (`pg_dump`) and back up the `media` volume (the actual documents). Those two together are a complete restore.

## Example Use

Paperless-ngx can be used to organize and track:

- scanned mail, contracts, invoices and receipts
- full-text search across everything via OCR (including German + English)
- automatic tagging, correspondents and document types with matching rules
- a watched "consume" folder — drop a PDF in and it's ingested automatically
- email ingestion (pull attachments straight from a mailbox)
- a paperless home archive that replaces a filing cabinet

This gives an individual or household a private, searchable document archive instead of folders of loose paper and PDFs.

## Thanks

Thanks to the Paperless-ngx project and its community of maintainers for a powerful, actively developed, open-source document management system that makes a paperless home archive genuinely practical.

## Links

- Source: https://github.com/paperless-ngx/paperless-ngx
- Documentation: https://docs.paperless-ngx.com
