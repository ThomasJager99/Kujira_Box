# Excalidraw

Excalidraw is a **virtual whiteboard with a hand-drawn feel** — quick sketches,
flowcharts, wireframes, architecture diagrams, mind maps. It's fast, it looks
friendly (that deliberate "drawn by hand" style), and it exports cleanly to PNG,
SVG or its own `.excalidraw` files. Self-hosting gives you an always-available
scratchpad on your own domain, with nothing sent to anyone else's servers.

The self-hosted container is the **static frontend**: Excalidraw runs entirely
in your browser and saves your current canvas to the browser's **local
storage**. There's no server-side database — the container just serves the app,
which makes it tiny and stateless.


<p align="center">
  <img src="../../../assets/logos/excalidraw.svg" width="180">
</p>

<br>


## Official container image

[`excalidraw/excalidraw`](https://hub.docker.com/r/excalidraw/excalidraw) —
official build of [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw).

> **Pinning:** Excalidraw's Docker tags aren't clean semver, so `latest` is the
> practical default. For a reproducible deploy, pin by **digest** from Docker
> Hub instead:
> ```
> image: excalidraw/excalidraw@sha256:<digest-from-docker-hub>
> ```

## Deployment Notes

- **Stateless + browser-local storage.** Your drawing is saved in *your
  browser*, per device — not on the server. Clearing site data loses it; use the
  in-app *Save to file* / *Export* for anything you want to keep, or load it on
  another device.
- **Live collaboration is a separate piece.** The real-time multi-user mode
  (shareable session links, cursors) needs the
  [`excalidraw-room`](https://github.com/excalidraw/excalidraw-room) websocket
  server alongside this. It's optional and not included here — solo use and
  local saving work without it. Add it later if you want shared sessions.
- **Port:** app on container `80`, published as `${EXCALIDRAW_PORT}` (default
  `5090`). Set `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your reverse
  proxy.
- **Great hardening candidate.** Being a stateless static site, it takes
  `read_only: true` cleanly (the compose shows the Level-2 block, commented) —
  nginx just needs a few tmpfs scratch paths and `NET_BIND_SERVICE` to bind port
  80 (hardening entry 03).

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — LISTEN_ADDR, port, (optionally pin a digest)
docker compose up -d
# open http://<host>:5090 and start drawing
```

## Thanks

Thanks to the [Excalidraw](https://github.com/excalidraw/excalidraw) team for a
delightful, genuinely useful open-source whiteboard.

## Links

- Project: https://excalidraw.com/
- Source: https://github.com/excalidraw/excalidraw
- Collaboration server: https://github.com/excalidraw/excalidraw-room
