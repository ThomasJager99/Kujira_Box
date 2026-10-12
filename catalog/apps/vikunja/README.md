# Vikunja

**Vikunja** is a self-hosted to-do and task-management app — the open-source
answer to Todoist / Microsoft To Do. You organise work into projects and tasks,
then view them however you think best: as plain lists, a **kanban** board, a
table, a **Gantt** timeline or a **calendar**. Tasks support due dates,
reminders, priorities, labels, subtasks, assignees, attachments and recurring
schedules, and there are web, desktop and mobile clients plus a CalDAV endpoint
so your tasks show up in other calendar apps. Everything lives on your own
server, so your planning and personal data never sit in someone else's cloud.
For a homelab it's the "run my life/projects" app that replaces a paid task
service.


<p align="center">
  <img src="../../../assets/logos/vikunja.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `vikunja/vikunja` (unified frontend + API in one image)
- **Project:** https://github.com/go-vikunja/vikunja
- **Docs:** https://vikunja.io/docs/

## Deployment Notes

- **Two containers:** `vikunja` (the app — single HTTP interface on `3456`) and
  `db` (**PostgreSQL 16**). The app can also use MySQL or embedded SQLite; here
  it's Postgres for a proper multi-container example.
- **Startup order.** The app waits for `db` to be `service_healthy`.
- **Public URL.** `VIKUNJA_SERVICE_PUBLICURL` must match how you actually reach
  it (scheme + host + port), or links and the API/CORS behave oddly.
- **Session secret.** `VIKUNJA_SERVICE_JWTSECRET` signs logins — set a strong
  value (`openssl rand -hex 32`) and keep it stable.
- **Persistent state.** Named volumes `vikunja_files` (uploaded attachments) and
  `db_data` (PostgreSQL). Back up `db` with `pg_dump` plus the files volume.
- **Healthchecks:** app → `/api/v1/info` via busybox `wget`; db → `pg_isready`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

### House-style upgrades used here

This stack showcases the upgraded template:

- **Pinned versions.** `.env` pins `VIKUNJA_VERSION` and `POSTGRES_VERSION` to
  known tags instead of floating on `:latest`, so a rebuild is reproducible.
  Rule of thumb: `latest` is fine for throwaway/stateless tools, but pin
  anything that owns data (databases, this app).
- **Network isolation.** Two networks: `frontend` (faces the published port and
  the internet) and `backend` marked `internal: true` (no internet route). The
  app sits on both; the **database sits on `backend` only**, so it's reachable
  by the app but not from outside the stack and can't phone home.
- **Optional hardening** (commented in the compose): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable per service after testing, since
  some apps misbehave on a read-only root filesystem.
- **Secrets.** `.env` in plaintext is fine for a homelab. For production-grade
  secret handling, move them to **Docker secrets** or **SOPS + age** (encrypted
  secrets in git) instead of a plaintext env file.

## Example Use

- Run personal projects and daily tasks with reminders and recurring to-dos.
- Switch between list, kanban, table, Gantt and calendar views of the same work.
- Subscribe to your tasks in another calendar app via CalDAV.
- Share projects and assign tasks if you use it with others.

## Thanks

To the **Vikunja** project and its maintainers for a full-featured, genuinely
self-hostable task manager.

## Links

- Project: https://github.com/go-vikunja/vikunja
- Documentation: https://vikunja.io/docs/
- Config reference: https://vikunja.io/docs/config-options/
