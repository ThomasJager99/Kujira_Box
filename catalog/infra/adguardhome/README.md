# AdGuard Home

**AdGuard Home** is a network-wide DNS server that blocks ads, trackers and
malicious domains for **every device on your network at once** — no browser
extension, no per-device setup. You point your router (or individual devices) at
AdGuard Home as their DNS resolver; it then filters every lookup against
blocklists, answers the rest, and gives you a clean web dashboard with live
query logs, per-client rules, statistics and optional parental controls. It can
also run encrypted DNS (DoH/DoT), act as your local DNS (resolve
`service.home` names), and even serve DHCP. For a homelab it's one of the
highest-impact single containers you can run: install it once and the whole
house gets faster, quieter, more private browsing.


<p align="center">
  <img src="../../../assets/logos/adguard-home.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `adguard/adguardhome`
- **Project:** https://github.com/AdguardTeam/AdGuardHome
- **Docs:** https://github.com/AdguardTeam/AdGuardHome/wiki

## Deployment Notes

- **One container.** Single lightweight `adguardhome` service (Go, Alpine-based).
- **Persistent state.** Two named volumes: `adguard_work`
  (`/opt/adguardhome/work` — runtime data, query log, statistics) and
  `adguard_conf` (`/opt/adguardhome/conf` — the `AdGuardHome.yaml` config). Back
  up `conf` to keep your rules and settings.
- **Ports.** `53/tcp`+`53/udp` is the DNS port your clients use. `3000` serves
  the **first-run setup wizard**; during that wizard set the **Admin Web
  Interface to port 3000** too, so this stack exposes a single UI port. DoT
  (`853`), DoH (`443`) and DHCP (`67`/`68`) are commented — enable only what you
  need.
- **Port 53 conflict (important on Linux).** Many distros already run a stub
  resolver on `53` (`systemd-resolved`). If the container fails to bind `53`,
  free it first: disable `DNSStubListener` in
  `/etc/systemd/resolved.conf` (`DNSStubListener=no`) and repoint
  `/etc/resolv.conf`, then restart. Or publish DNS on a dedicated host IP.
- **Healthcheck** fetches the web port with the image's busybox `wget`.
- **Resource limits** (`mem_limit` / `mem_reservation` / `cpus`) and all
  environment-specific values are externalized to `.env`.
- **Alternative:** Pi-hole covers the same job; AdGuard Home is a single
  container with DoH/DoT built in, which is why it's used here.

## Example Use

- Block ads and trackers for **every** device — phones, TVs, IoT — via one DNS.
- Define local DNS rewrites so `nas.home` / `grafana.home` resolve on your LAN.
- See exactly what each device is phoning home to in the live query log.
- Apply per-client rules (e.g. stricter filtering for the kids' devices).
- Serve encrypted DNS (DoH/DoT) so upstream lookups aren't snooped.

## Thanks

To the **AdGuard** team for making network-wide, self-hostable DNS filtering
this approachable.

## Links

- Project: https://github.com/AdguardTeam/AdGuardHome
- Wiki / docs: https://github.com/AdguardTeam/AdGuardHome/wiki
- Docker notes: https://github.com/AdguardTeam/AdGuardHome/wiki/Docker
