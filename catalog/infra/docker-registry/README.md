# Private Docker Registry + UI

Run your **own container registry** — a private place to `docker push` and
`docker pull` your images, instead of relying on Docker Hub. Handy for your own
builds, for your CI (pairs nicely with Woodpecker), for keeping images on your
network, and for avoiding Hub's rate limits. This stack adds a clean **web UI**
(Joxit) on top so you can browse repositories, tags and sizes, see digests, and
delete images from the browser.

Two containers: the **registry** (Distribution v3 — the real storage backend)
and the **UI**, whose nginx also *proxies* the registry API. That means clients
push and pull through the UI's single port, and the registry container itself
stays unpublished on an internal, no-egress network.



<p align="center">
  <img src="../../../assets/logos/container-registry-svgrepo-com.svg" width="180">
</p>

<br>


## Official container images

- Registry: [`registry`](https://hub.docker.com/_/registry) (official
  [distribution/distribution](https://github.com/distribution/distribution))
- UI: [`joxit/docker-registry-ui`](https://github.com/Joxit/docker-registry-ui)

Pin both (here `registry:3.1.2`, `joxit/docker-registry-ui:2.6.0`).

## Deployment Notes

### 1. Create the login (htpasswd) — do this first

The registry is configured to **require authentication**. Create a bcrypt
htpasswd file before starting (uses the `httpd` image so you don't need
`htpasswd` installed locally):

```bash
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2 -Bbn myuser 'mypassword' > auth/htpasswd
```

(`-B` = bcrypt, required by the registry.) Add more users by appending more
lines. Keep `auth/` out of git.

### 2. TLS — the registry's one hard rule

Docker **refuses to push to a remote registry over plain HTTP.** Two options:

- **Recommended:** put this behind your reverse proxy with a real TLS cert, and
  push to `https://registry.example.com`. Nothing else to configure on clients.
- **LAN / testing without TLS:** add the host to each client's
  `/etc/docker/daemon.json` and restart Docker:
  ```json
  { "insecure-registries": ["registry.lan:5080"] }
  ```
  Use this only on a trusted network.

### 3. Exposure

- Only the **UI** publishes a port (`${REGISTRY_UI_PORT}`, default `5080`); the
  registry has no `ports:` and lives on an `internal` network.
- Set `LISTEN_ADDR=127.0.0.1` in `.env` to keep even the UI host-only and reach
  it via your reverse proxy — then the only thing on the LAN is 80/443.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# create auth/htpasswd (step 1 above), edit .env (pin versions, LISTEN_ADDR)
docker compose up -d

# log in and push through the UI's port:
docker login <host>:5080
docker tag nginx:alpine <host>:5080/my/nginx:alpine
docker push <host>:5080/my/nginx:alpine
# browse it at http://<host>:5080
```

### Security note

- **Auth is on by default** (htpasswd) — don't run a registry open; anyone who
  can reach it can push/pull your images.
- **Registry is internal + no-egress** (hardening entry 06): it only stores
  images, so it has no route to the internet and isn't reachable except through
  the UI proxy.
- **Bind address is configurable** (hardening entry 07): default `0.0.0.0` so it
  works out of the box; set `127.0.0.1` for a proxied, host-only deploy.

## Thanks

Thanks to the [Distribution](https://github.com/distribution/distribution)
project (the registry) and to [Joxit](https://github.com/Joxit/docker-registry-ui)
for the UI.

## Links

- Registry docs: https://distribution.github.io/distribution/
- Registry config reference: https://distribution.github.io/distribution/about/configuration/
- UI: https://github.com/Joxit/docker-registry-ui
