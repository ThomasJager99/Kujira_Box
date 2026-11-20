# Metabase

Metabase is **open-source business intelligence** — you connect it to your
databases and build charts, dashboards and "questions" over your data, with a
friendly point-and-click query builder (and raw SQL when you want it). It's the
tool for *making sense* of data you already have: turn an app's Postgres into a
dashboard, track metrics over time, schedule emailed reports, set alerts on
thresholds. Think of it as the readable, shareable front-end to your databases.

Two services here: **Metabase** and a **PostgreSQL "application database."**
Important distinction — that Postgres stores *Metabase's own* configuration
(your saved questions, dashboards, users), **not** the data you analyze. You
connect Metabase to *those* databases from inside its UI after setup.


<p align="center">
  <img src="../../../assets/logos/metabase.svg" width="180">
</p>

<br>


## Official container images

- App: [`metabase/metabase`](https://hub.docker.com/r/metabase/metabase)
  ([metabase/metabase](https://github.com/metabase/metabase)) — the OSS edition,
  tags like `v0.65.6`.
- DB: [`postgres`](https://hub.docker.com/_/postgres) (official)

Pin both (here `metabase/metabase:v0.65.6`, `postgres:16-alpine`).

## Deployment Notes

- **Use an external app DB.** Metabase defaults to an embedded H2 file database
  that is **not** safe for real use (corruption risk, no good backups). This
  stack points it at Postgres via the `MB_DB_*` variables — the right setup from
  day one.
- **It's a JVM app — cap the heap.** `JAVA_OPTS=-Xmx1g` keeps it inside
  `mem_limit`. Without it the JVM sizes to the whole host and gets OOM-killed
  (hardening entry 05). Give it more (`-Xmx2g` + a higher `mem_limit`) if you
  load big dashboards.
- **Slow first boot.** Initial startup runs DB migrations and can take a minute
  or two — the healthcheck has a 90s `start_period` for that. Don't panic if
  `/api/health` isn't green immediately.
- **First run:** open the UI and the setup wizard creates the admin account and
  lets you add your first data source (a database you want to analyze).
- **Port:** app on container `3000`, published as `${METABASE_PORT}`. Set
  `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse proxy.
- **Backups:** everything Metabase knows lives in the `metabase-db` volume —
  that's your single backup target.

### Security note

- The app DB is on an **internal, no-egress network** (hardening entry 06).
- When you connect Metabase to *your* databases, prefer a **read-only** DB user
  for each source — Metabase only needs to read to build charts, and a
  read-only account means a mistake (or a compromise) can't modify your data.
- Bind address configurable (entry 07); put it behind reverse proxy + TLS
  (entry 09), and consider forward-auth/SSO (entry 10) if exposed.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# generate a DB password:  openssl rand -hex 24
# edit .env — set POSTGRES_PASSWORD, LISTEN_ADDR
docker compose up -d
# open http://<host>:3000 → run the setup wizard → connect a database to analyze
```

## Thanks

Thanks to the [Metabase](https://github.com/metabase/metabase) team for making
BI genuinely accessible and open-source.

## Links

- Docs: https://www.metabase.com/docs/latest/
- Running on Docker: https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker
- Source: https://github.com/metabase/metabase
