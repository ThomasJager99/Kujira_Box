# Speedtest Tracker

Speedtest Tracker runs **Ookla Speedtest on a schedule you set** and keeps a
history of your internet connection: download, upload, ping and jitter charted
over time, with a clean dashboard, an API, and alerting (notify when results
drop below a threshold). It's the tool that lets you *prove* to your ISP that
the line you pay for isn't the line you're getting — and spot when things
quietly degrade.

It's a single container with an embedded **SQLite** database by default, so
there's nothing else to run. (It can use MySQL/MariaDB/Postgres instead for
bigger setups — not needed for a home line.)


<p align="center">
  <img src="../../../assets/logos/speedtest-tracker.png" width="180">
</p>

<br>


## Official container image

[`ghcr.io/alexjustesen/speedtest-tracker`](https://github.com/alexjustesen/speedtest-tracker)
— the author's own image (there's also a LinuxServer build). Pin a release tag
(here `v1.15.1`) rather than `latest`.

## Deployment Notes

- **`APP_KEY` is required** and must be stable. Generate it once with
  `echo "base64:$(openssl rand -base64 32)"` and keep the same value forever —
  it encrypts stored settings, so changing it later breaks them.
- **`APP_URL`** should match how you actually reach the dashboard (behind a
  reverse proxy, your `https://...` host). It's used for asset and link URLs.
- **Scheduling:** `SPEEDTEST_SCHEDULE` is a cron expression — hourly by default.
  Don't go *too* aggressive; each test uses real bandwidth. Hourly or every few
  hours is plenty for trend-watching.
- **Port:** dashboard on container `80`, published as `${SPEEDTEST_PORT}`
  (default `8765`).
- **First run:** open the dashboard and create your admin user, then configure
  notifications/thresholds under *Settings*.
- **It needs internet egress** — obviously, since it measures your line. That's
  why (see security below) it can't be put on a no-egress internal network.

### Security note

This stack is a deliberate example in the hardening rubric's
**network-segmentation** entry: it's the service that **must** keep egress, so
you *can't* lock it down the way you would a database. Instead it leans on the
other layers — pinned image from the author's own registry, `no-new-privileges`,
memory/CPU/PID limits, and a reverse proxy + TLS in front. The heavier
`read_only` / `cap_drop` block is left commented because the image writes as
root at startup.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# generate APP_KEY:  echo "base64:$(openssl rand -base64 32)"
# edit .env — paste APP_KEY, set APP_URL, TZ, schedule
docker compose up -d
# open http://<host>:8765 → create admin user → wait for the first scheduled test
```

## Thanks

Thanks to [Alex Justesen](https://github.com/alexjustesen/speedtest-tracker)
for Speedtest Tracker, and to Ookla for the Speedtest CLI it builds on.

## Links

- Docs: https://docs.speedtest-tracker.dev/
- Source: https://github.com/alexjustesen/speedtest-tracker
- Image: https://github.com/alexjustesen/speedtest-tracker/pkgs/container/speedtest-tracker
