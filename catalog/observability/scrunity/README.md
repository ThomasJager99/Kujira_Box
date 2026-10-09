# Scrutiny

**Scrutiny** is a web dashboard for your disks' **S.M.A.R.T.** health. Hard
drives and SSDs constantly self-report metrics — temperature, power-on hours,
reallocated sectors, wear levelling, read/write error rates — but raw
`smartctl` output is cryptic and nobody checks it until a disk has already
failed. Scrutiny collects those metrics on a schedule, stores their history,
and shows each drive as a clear "passed / warning / failed" card with trend
graphs, so you see a disk *starting* to go bad while there's still time to
replace it and move your data. It layers real-world failure research (Backblaze
stats) on top of the manufacturer thresholds for smarter warnings. For a homelab
where your data sits on spinning rust or aging SSDs, it's cheap insurance.


<p align="center">
  <img src="../../../assets/logos/scrutiny.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `ghcr.io/analogj/scrutiny` (`master-omnibus` = all-in-one)
- **Project:** https://github.com/AnalogJ/scrutiny
- **Docs:** https://github.com/AnalogJ/scrutiny/tree/master/docs

## Deployment Notes

- **One container (omnibus).** Bundles the web UI, the metrics collector and
  **InfluxDB** together — the simplest setup for a single host. Large/multi-host
  fleets can run the web and collector images separately instead.
- **Needs raw disk access.** Reading S.M.A.R.T. data requires:
  - `cap_add: SYS_RAWIO` (and `SYS_ADMIN` for most **NVMe** drives),
  - each physical disk passed under `devices:` (match your machine — check with
    `lsblk`; remove entries that don't exist or the container errors),
  - `/run/udev` mounted read-only so Scrutiny can read device metadata.
  - If SMART reads fail on NVMe, try removing `no-new-privileges`.
- **Single HTTP interface** on port `8080`.
- **Scheduled scans.** `COLLECTOR_CRON_SCHEDULE` controls how often the
  collector polls your drives (default: daily at 01:00).
- **Persistent state.** Named volumes `scrutiny_config`
  (`/opt/scrutiny/config`) and `scrutiny_influxdb` (`/opt/scrutiny/influxdb` —
  the metrics history).
- **Healthcheck** hits the `/api/health` endpoint.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Catch a drive with rising reallocated-sector counts before it dies.
- Watch disk temperatures across the whole box in one dashboard.
- Keep a history of wear/health trends for every drive.
- Pair with notifications (Scrutiny supports webhooks/email) so a failing disk
  pings you — e.g. route alerts to your ntfy server.

## Thanks

To **AnalogJ** and the Scrutiny contributors for turning cryptic S.M.A.R.T. data
into something you'll actually look at.

## Links

- Project: https://github.com/AnalogJ/scrutiny
- Documentation: https://github.com/AnalogJ/scrutiny/tree/master/docs
- Notifications: https://github.com/AnalogJ/scrutiny/blob/master/docs/NOTIFICATIONS.md
