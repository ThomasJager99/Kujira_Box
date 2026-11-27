# Grist

Grist is a **spreadsheet-database hybrid** — it looks like a spreadsheet, but
underneath is a real relational engine: typed columns, references between tables,
Python formulas, access rules, and custom page layouts. Think "Airtable you host
yourself," but more serious about data: it's genuinely useful for trackers,
inventories, budgets, and small apps where plain cells aren't enough but a full
database is overkill.

A single container, SQLite-backed (each document is its own SQLite file under
`/persist`) — no external database to run.


<p align="center">
  <img src="../../../assets/logos/grist.svg" width="180">
</p>

<br>


## Official container image

[`gristlabs/grist`](https://hub.docker.com/r/gristlabs/grist) — the project's own
image (full edition; `gristlabs/grist-oss` is the pure open-source build). Pin a
release tag (here `1.7.22`).

## Deployment Notes

- **Generate two secrets.** `GRIST_SESSION_SECRET` (signs sessions) and
  `GRIST_BOOT_KEY` (protects the admin panel) — both `openssl rand -hex 32`.
  Setting `GRIST_BOOT_KEY` yourself avoids hunting for the boot key Grist
  otherwise prints to the log on first start.
- **One volume to back up:** `grist-data:/persist` holds every document plus the
  home database. That's the whole instance.
- **Formula sandbox = a real security feature.** Grist runs the Python in your
  formulas inside a sandbox so a malicious or buggy formula can't escape. The
  default flavor is **gVisor** (a userspace kernel — see hardening entry 19);
  if it struggles under a locked-down container, `GRIST_SANDBOX_FLAVOR=pyodide`
  runs formulas in WebAssembly, which needs no special privileges.
- **Port:** app on `8484`, published as `${GRIST_PORT}`. Set
  `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse proxy, and set
  `GRIST_HOME_URL` to the public URL there.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — set GRIST_DEFAULT_EMAIL, generate the two secrets
docker compose up -d
# open http://<host>:8484 → log in as the admin email → create a document
```

## Thanks

Thanks to [Grist Labs](https://github.com/gristlabs/grist-core) for a genuinely
open, self-hostable spreadsheet-database.

## Links

- Project: https://www.getgrist.com/
- Source: https://github.com/gristlabs/grist-core
- Self-managed docs: https://support.getgrist.com/self-managed/
