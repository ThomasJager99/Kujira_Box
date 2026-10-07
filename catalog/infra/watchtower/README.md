# Watchtower

**Watchtower** keeps your running containers up to date automatically. On a
schedule it checks whether a newer image has been published for each container
it watches, and when one has, it pulls it, gracefully stops the old container
and starts a new one with the **exact same** configuration (same flags, volumes,
env, networks). No more logging in to pull images by hand. It can clean up the
old images afterwards, restart containers one at a time to minimise downtime,
and send you a notification about what it changed. For a homelab it's the
"set-and-forget" piece that keeps your stack patched — used deliberately, with
opt-in labels, so updates stay under your control.


<p align="center">
  <img src="../../../assets/logos/watchtower.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `containrrr/watchtower`
- **Project:** https://github.com/containrrr/watchtower
- **Docs:** https://containrrr.dev/watchtower/

## Deployment Notes

- **One container.** Single `watchtower` service — no web UI, no database, no
  ports. It just runs on a schedule in the background.
- **Docker socket (read-write).** Mounts `/var/run/docker.sock` **RW** — unlike
  a log viewer, Watchtower must *pull images and recreate containers*, so it
  needs write access. That equals host-root power: run only this one trusted
  image against it, keep the host locked down, and consider a
  docker-socket-proxy that exposes just the needed endpoints.
- **Opt-in by default.** `WATCHTOWER_LABEL_ENABLE=true` means it only touches
  containers you explicitly label. Add this to any container you want
  auto-updated:
  ```yaml
  labels:
    - "com.centurylinklabs.watchtower.enable=true"
  ```
  Set `WATCHTOWER_LABEL_ENABLE=false` to watch **every** container instead — convenient,
  but auto-updating everything can pull a breaking change unattended, so pin or
  exclude anything stateful/critical.
- **Schedule.** `WATCHTOWER_SCHEDULE` is a 6-field cron (default: daily at
  04:00). `WATCHTOWER_CLEANUP` removes superseded images; `WATCHTOWER_ROLLING_RESTART`
  updates containers one at a time.
- **Monitor-only.** Set `WATCHTOWER_MONITOR_ONLY=true` to be *notified* about
  updates without applying them — a safe way to stay informed and update by hand.
- **Notifications.** Point `WATCHTOWER_NOTIFICATION_URL` at a shoutrrr target —
  e.g. your **ntfy** server — to get a push when something updates.
- **Healthcheck** uses Watchtower's built-in `--health-check` subcommand.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Auto-patch your stateless, frequently-updated services (dashboards, tools)
  overnight.
- Run in monitor-only mode to get a nightly "updates available" push and decide
  what to update yourself.
- Pair with **ntfy** so every applied update pings your phone.
- Label just a handful of containers so critical/stateful ones are never touched
  automatically.

## Thanks

To the **Watchtower / containrrr** maintainers for a reliable, no-fuss container
auto-updater.

## Links

- Project: https://github.com/containrrr/watchtower
- Documentation: https://containrrr.dev/watchtower/
- Notifications (shoutrrr): https://containrrr.dev/watchtower/notifications/
