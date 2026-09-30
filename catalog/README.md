# Docker Compose Services


<p align="center">
  <img src="../assets/mascot/termkujira.png" width="180">
</p>

<br>


This directory collects the self-hosted services running in the homelab. Each
service is a self-contained Docker Compose stack — its own `docker-compose` file,
an example env and a README — and the stacks are grouped by purpose into the
categories below. The collection grows over time; the current list of services
in each category lives in that category's README.

Nothing here contains real credentials — every stack ships an example env you
copy and fill in.

<table>
  <tr>
    <td width="80" align="center"><a href="media/"><img src="../assets/icons/clapperboard.svg" width="64" alt="Media"></a></td>
    <td><a href="media/"><b>Media</b></a><br>Streaming and libraries for video, audiobooks, comics and photos.</td>
  </tr>
  <tr>
    <td width="80" align="center"><a href="apps/"><img src="../assets/icons/app-window.svg" width="64" alt="Apps"></a></td>
    <td><a href="apps/"><b>Apps</b></a><br>Personal and productivity applications — finance, documents, notes, tasks, passwords.</td>
  </tr>
  <tr>
    <td width="80" align="center"><a href="infra/"><img src="../assets/icons/server.svg" width="64" alt="Infrastructure"></a></td>
    <td><a href="infra/"><b>Infrastructure</b></a><br>The plumbing — reverse proxy, dashboard, stack manager and Git hosting.</td>
  </tr>
  <tr>
    <td width="80" align="center"><a href="observability/"><img src="../assets/icons/activity.svg" width="64" alt="Observability"></a></td>
    <td><a href="observability/"><b>Observability</b></a><br>Metrics, logs, uptime and live system health.</td>
  </tr>
  <tr>
    <td width="80" align="center"><a href="databases/"><img src="../assets/icons/database-zap.svg" width="64" alt="Databases"></a></td>
    <td><a href="databases/"><b>Databases</b></a><br>Shared and standalone data backends.</td>
  </tr>
</table>

## Layout

```
docker_compose_services/
├── media/           video, audiobooks, comics, photos
├── apps/            personal & productivity apps
├── infra/           proxy, dashboard, stack manager, git
├── observability/   metrics, logs, uptime
└── databases/       data backends
```

Each service folder keeps its own `docker-compose` file, example env and README,
so it can be understood — and run — on its own. New services are added into the
category they belong to, and listed in that category's README.

---

## Requirements

Every stack here needs **Docker Engine** and the **Docker Compose v2** plugin
(`docker compose`, with a space). The run commands are identical on any Linux
distro — only the install step differs.

Quickest cross-distro install (detects your distro):

```bash
curl -fsSL https://get.docker.com | sh
```

Or install from your distro's packages:

| Distro | Install |
|--------|---------|
| Debian / Ubuntu | Docker's apt repo → `apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin` |
| Fedora / RHEL | Docker's dnf repo → `dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin` |
| Arch | `pacman -S docker docker-compose` |


Prefer Docker's official repo so you get Compose **v2**. Some older setups — e.g.
Debian 12 with the distro's own `docker-compose` package — only ship the legacy
**v1** standalone, where the command is `docker-compose` (with a hyphen). It is
otherwise identical: just swap `docker compose` → `docker-compose` in the steps.

---

## Services

### [Apps](apps/)
- [Firefly III](apps/firefly/) — personal finance manager (accounts, budgets, reports)
- [Paperless-ngx](apps/paperless-ngx/) — document management with OCR & full-text search
- [Portainer](apps/portainer/) — web UI for managing Docker
- [SillyTavern](apps/sillytavern/) — front-end for chatting with LLMs / AI characters
- [Stirling-PDF](apps/stirling-pdf/) — web toolkit for PDF operations (merge, split, OCR, convert)
- [Trilium](apps/trilium/) — hierarchical note-taking / knowledge base
- [Vaultwarden](apps/vaultwarden/) — password manager server (Bitwarden-compatible)
- [Wekan](apps/wekan/) — kanban board (Trello alternative)
- [Linkwarden](apps/linkwarden/) — bookmark manager that archives pages (screenshot, PDF, full text) and searches them
- [Mealie](apps/mealie/) — recipe manager & meal planner (imports recipes from a URL)
- [IT-Tools](apps/it-tools/) — ~100 handy developer/IT utilities in the browser
- [ntfy](infra/ntfy/) — push notifications to your phone from scripts & servers

### [Media](media/)
- [Audiobookshelf](media/audiobookshelf/) — audiobook & podcast server
- [Immich](media/immich/) — photo & video backup with ML search (Google Photos alternative)
- [Jellyfin](media/jellyfin/) — media server for movies, shows & music
- [Kavita](media/kavita/) — reader/server for ebooks, comics & manga
- [Komga](media/komga/) — comics, manga & digital book library server
- [Navidrome](media/navidrome/) — music server (Subsonic-compatible) with offline apps

### [Infra](infra/)
- [Dockge](infra/dockge/) — web manager for Docker Compose stacks
- [Gitea](infra/gitea/) — self-hosted Git service (repos, issues, CI)
- [Homepage](infra/homepage/) — dashboard / start page for all services
- [Nginx](infra/nginx/) — reverse proxy / web server
- [Transmission VPN hub](infra/transmission-vpn-hub/) — VPN gateway (gluetun) + Transmission; route any service through the VPN
- [Nginx Proxy Manager](infra/nginx-proxy-manager/) — reverse proxy UI with automatic Let's Encrypt HTTPS

### [Observability](observability/)
- [Glances](observability/glances/) — live system resource monitor
- [Monitoring](observability/monitoring/) — Prometheus + Grafana + Loki metrics & logs stack
- [Uptime Kuma](observability/uptime_kuma/) — uptime monitoring & status pages

### [Databases](databases/)
- [MySQL](databases/mysql/) — MySQL/MariaDB instance for development & learning
- [MySQL (prod)](databases/mysql_prod/) — production MySQL/MariaDB instance
- [Nginx + MySQL + Redis](databases/nginx_mysql_redis/) — bundled backend stack (proxy + database + cache)

