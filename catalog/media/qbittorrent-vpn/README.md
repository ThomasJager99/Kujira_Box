# qBittorrent (VPN-routed)

**qBittorrent** is a lightweight, open-source, ad-free BitTorrent client with a
full **web UI** — you manage torrents from the browser, with categories, RSS
auto-download, bandwidth scheduling and a built-in search. This stack ships it
the **safe way: routed entirely through a VPN** using **gluetun** as the gateway,
with a **kill-switch** so the client can *only* talk to the internet through the
encrypted tunnel. If the VPN connection drops, gluetun blocks all traffic — your
real IP address is never exposed while torrenting. It's the same VPN-hub pattern
as the Transmission stack, applied to qBittorrent.

> The client is just software; what you download is your responsibility. Routing
> through a VPN protects your privacy and is standard practice; it doesn't make
> infringing downloads legal. Use it for your own/legal content.


<p align="center">
  <img src="../../../assets/logos/qbittorrent.svg" width="180">
</p>

<br>


## Official container images

- **qBittorrent:** `lscr.io/linuxserver/qbittorrent` (LinuxServer.io)
- **gluetun (VPN):** `qmcgaw/gluetun`
- **Projects:** https://github.com/qbittorrent/qBittorrent · https://github.com/qdm12/gluetun

## Deployment Notes

- **Two containers, one network.** `gluetun` is the VPN gateway; `qbittorrent`
  runs with `network_mode: "service:gluetun"`, so **all** its traffic exits
  through the VPN. qBittorrent waits for gluetun to be `service_healthy`.
- **Ports live on gluetun.** Because qBittorrent shares gluetun's network, it
  **publishes no ports of its own** — the Web UI (`8080`) and torrent port
  (`6881`) are published on **gluetun**. `WEBUI_PORT` on qBittorrent must match
  the published Web UI port.
- **Kill-switch + LAN access.** gluetun blocks everything outside the tunnel by
  default; `FIREWALL_OUTBOUND_SUBNETS` must list your LAN subnet so you can still
  open the Web UI from your network.
- **VPN config.** Set your provider in `VPN_SERVICE_PROVIDER` (Mullvad,
  ProtonVPN, …) with a WireGuard key, **or** use `custom` to point gluetun at
  **your own WireGuard** (e.g. your RB5009 + VPS tunnel). See the gluetun wiki
  for per-provider variables.
- **Verify the tunnel** after starting: `docker exec qbittorrent wget -qO- ifconfig.me`
  should return the **VPN**'s IP, not yours.
- **First login.** Recent LinuxServer images generate a **temporary admin
  password** on first start — find it in `docker logs qbittorrent`, log in, then
  set your own password in the Web UI.
- **Why no network-split here.** qBittorrent uses gluetun's network namespace, so
  the upgraded template's `frontend`/`backend` split doesn't apply.
- **Persistent state.** Named volumes `gluetun_config`, `qbt_config` (`/config`)
  and a bind-mounted `/downloads`. Back up `qbt_config`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`. Pin `GLUETUN_VERSION` / `QBT_VERSION` for reproducible rebuilds.

## Example Use

- Download/seed over a VPN with a kill-switch so your real IP never leaks.
- Manage torrents from a browser anywhere on your LAN.
- Auto-download via RSS feeds; feed completed files to Sonarr/Radarr.
- Use your own WireGuard tunnel (custom provider) instead of a commercial VPN.

## Thanks

To the **qBittorrent** project and **qdm12** (gluetun), plus **LinuxServer.io**
for the maintained image.

## Links

- qBittorrent: https://github.com/qbittorrent/qBittorrent
- gluetun: https://github.com/qdm12/gluetun · wiki: https://github.com/qdm12/gluetun-wiki
- Image: https://docs.linuxserver.io/images/docker-qbittorrent
