# Home Assistant

**Home Assistant** is a local-first home automation platform — the open-source
hub that ties your smart home together. It connects to thousands of devices and
services (lights, switches, sensors, thermostats, cameras, vacuums, media
players, energy meters…) through a huge library of integrations, presents them
in one dashboard, and lets you build **automations** ("when motion after sunset →
turn on the hall light", "if nobody home → set thermostat back"). Crucially it
runs **on your own hardware and works offline** — your home's state and history
stay local, not in a vendor cloud, and most integrations keep working even
without internet. For a homelab it's the flagship "control my physical world"
app.


<p align="center">
  <img src="../../../assets/logos/home-assistant.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `ghcr.io/home-assistant/home-assistant` (the "Container" install)
- **Project:** https://github.com/home-assistant/core
- **Docs:** https://www.home-assistant.io/installation/

> **Container vs OS install.** This is Home Assistant **Container**, which does
> *not* include the Add-on store / Supervisor. If you want one-click add-ons
> (Mosquitto, Zigbee2MQTT, backups, etc.), use Home Assistant OS on a dedicated
> box/VM instead. For a Docker homelab, Container is the right fit.

## Deployment Notes

- **One container, host networking.** Runs with `network_mode: host` so it can
  **auto-discover** devices via mDNS/zeroconf, DHCP, UPnP and Bluetooth. The UI
  is served on the **host's port `8123`**.
- **Why no network isolation here.** With host networking Docker ignores
  `ports:` and you can't attach custom bridge networks — so the upgraded
  template's `frontend`/`backend` split doesn't apply to HA. (To change the
  port, set `http.server_port` in `configuration.yaml`.)
- **Persistent state.** Named volume `ha_config` (`/config`) holds *everything*:
  your configuration, automations, the recorder database and history. Back it
  up — this is the heart of your setup.
- **Hardware radios.** For Zigbee/Z-Wave USB sticks, uncomment the `devices:`
  block and point it at your adapter (`/dev/ttyUSB0`, `/dev/ttyACM0`). Bluetooth
  works via the mounted `/run/dbus`.
- **Pinned version.** HA uses **calendar versions** (e.g. `2026.10.2`) and ships
  breaking changes regularly — pin exactly and read release notes before
  upgrading. Verified latest at build time: `2026.10.2`.
- **Healthcheck** fetches `/manifest.json` on `8123` with the image's busybox
  `wget`.
- **Optional hardening** (commented): `cap_drop: [ALL]`, `read_only: true`,
  `tmpfs: [/tmp]` — enable after testing, as some integrations need extra access.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.
- **First run.** Open `http://<host>:8123` and complete onboarding (create your
  account, name your home, let it discover devices).

## Example Use

- Control all your smart devices from one local dashboard and mobile app.
- Automate routines by time, presence, sensor state or sun position.
- Keep energy/temperature/history data locally, no vendor cloud.
- Integrate cameras, media players and notifications (e.g. push to your ntfy).

## Thanks

To the **Home Assistant** project and its enormous community for a powerful,
privacy-respecting, local-first smart-home platform.

## Links

- Project: https://github.com/home-assistant/core
- Installation (Container): https://www.home-assistant.io/installation/linux#docker-compose
- Integrations: https://www.home-assistant.io/integrations/
