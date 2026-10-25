# BookStack

**BookStack** is a self-hosted platform for organising and storing information —
a wiki / documentation tool with a refreshingly simple structure. Content is
arranged as **Shelves → Books → Chapters → Pages**, which maps naturally onto
how you actually think about documentation, and pages are written in a friendly
**WYSIWYG** editor (Markdown also supported). It has full-text search, page
revisions and history, cross-linking, image/attachment management, roles and
permissions, and export to PDF/HTML/Markdown. It's intentionally *not* a
sprawling enterprise wiki — it's clean and approachable. For a homelab it's the
ideal place to document your own setup: runbooks, configs, "how I built X", and
an ops journal.


<p align="center">
  <img src="../../../assets/logos/bookstack.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `lscr.io/linuxserver/bookstack` (LinuxServer.io)
- **Project:** https://github.com/BookStackApp/BookStack
- **Docs:** https://www.bookstackapp.com/docs/

## Deployment Notes

- **Two containers:** `bookstack` (the app, served on `80` → host `6875`) and
  `db` (**MariaDB 11**). BookStack requires a MySQL/MariaDB database.
- **Startup order.** The app waits for `db` to be `service_healthy`.
- **App URL.** `APP_URL` **must** match how you reach BookStack (scheme + host +
  port), or internal links, images and redirects break. Change it later only via
  this variable, then restart.
- **App key.** `APP_KEY` is a Laravel encryption key — generate once with
  `echo "base64:$(openssl rand -base64 32)"` and keep it stable (changing it
  invalidates existing encrypted data/sessions).
- **File ownership.** App runs under `PUID`/`PGID` (default `1000:1000`).
- **Persistent state.** Named volumes `bookstack_config` (`/config` — app config
  + uploaded images/attachments) and `db_data` (MariaDB). Back up both.
- **First login.** Default admin is `admin@admin.com` / `password` — change it
  immediately after first login.
- **Healthchecks:** app → `/login` via busybox `wget`; db → MariaDB's
  `healthcheck.sh --connect --innodb_initialized`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

### House-style upgrades used here

- **Pinned versions.** MariaDB pinned to `11`; pin a BookStack release tag too
  for reproducible rebuilds.
- **Network isolation.** `frontend` (published port + internet) and `backend`
  (`internal: true`). The **database is on `backend` only** — reachable by the
  app, invisible outside the stack, no internet egress.
- **Optional hardening** (commented on the app): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing (LSIO images often
  need a writable root).
- **Secrets.** `.env` plaintext is fine for homelab; for production move to
  Docker secrets or SOPS + age.

## Example Use

- Document your homelab: runbooks, service configs, network diagrams, decisions.
- Keep an ops journal (what you changed, why, and the result).
- Build a personal/team knowledge base with search and permissions.
- Export pages/books to PDF or Markdown for sharing or backup.

## Thanks

To **Dan Brown** and the BookStack contributors, and **LinuxServer.io** for the
maintained image.

## Links

- Project: https://github.com/BookStackApp/BookStack
- Documentation: https://www.bookstackapp.com/docs/
- Image: https://docs.linuxserver.io/images/docker-bookstack
