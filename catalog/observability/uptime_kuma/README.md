This directory contains an example deployment of Uptime Kuma, an open-source self-hosted monitoring tool used to watch whether your services are up. Uptime Kuma provides a clean web interface where you add monitors for websites, APIs, containers, TCP ports, DNS records and more, and it checks them on an interval, records response times, and alerts you the moment something goes down. It can also publish public or private status pages, so you (or your users) get a single at-a-glance view of everything's health — all running on your own server, with no external monitoring service involved.

## Image

<p align="center">
  <img src="../../../assets/logos/uptime-kuma.svg" width="180">
</p>

<br>

## Official container image

https://github.com/louislam/uptime-kuma — image published as `louislam/uptime-kuma`.

## Deployment Notes

- Runs as a single container — the whole app (checker, web UI and its embedded database) is self-contained.
- Stores all monitors, history and settings in a persistent Docker volume (`uptime_kuma_data` mounted at `/app/data`) that survives restarts and image upgrades.
- Exposes a single HTTP interface (container listens on port 3001); intended to sit behind a reverse proxy for TLS if reached over the open internet.
- Uses the image's built-in healthcheck binary (`extra/healthcheck`) rather than an external HTTP client, so the check is reliable regardless of which tools the image ships.
- Memory limit and reservation plus a CPU limit are set to keep it polite on a shared homelab host.
- Environment-specific values (image tag, host port, memory limit) are externalized to a `.env` file.
- No secrets in configuration: the admin account and all monitors are created through the web UI on first launch.

## Example Use

Uptime Kuma can be used to monitor and get alerted on:

- websites and web apps (HTTP/HTTPS status and response time)
- APIs and specific keyword/JSON responses
- TCP ports, ping (ICMP), DNS records and databases
- other self-hosted containers and services on the network
- public or private status pages summarizing everything's health
- notifications via Telegram, email, Discord, Slack, webhooks and 90+ others

This gives a single, private dashboard for knowing — and being told — when anything on the homelab goes down.

## Thanks

Thanks to Louis Lam and the Uptime Kuma project for building a polished, genuinely pleasant, open-source monitoring tool that is a joy to self-host.

## Links

- Source: https://github.com/louislam/uptime-kuma
- Website: https://uptime.kuma.pet
- Documentation: https://github.com/louislam/uptime-kuma/wiki
