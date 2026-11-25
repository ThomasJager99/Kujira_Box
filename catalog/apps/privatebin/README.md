# PrivateBin

PrivateBin is a **minimalist, open-source, encrypted pastebin** — a place to
paste text (a snippet, a log, an error, a password) and share it by link. The
twist: it's **zero-knowledge**. Your data is encrypted and decrypted **in your
browser**; the decryption key lives in the URL fragment (`#...`) which never
reaches the server, so the server only ever stores ciphertext and literally
*cannot* read your pastes. Add an expiry, optional password, or
burn-after-reading, and it's a genuinely private alternative to public paste
sites.

A single container (nginx + php-fpm), storing encrypted pastes on the
filesystem — no database to run.


<p align="center">
  <img src="../../../assets/logos/privatebin.svg" width="180">
</p>

<br>


## Official container image

[`privatebin/nginx-fpm-alpine`](https://hub.docker.com/r/privatebin/nginx-fpm-alpine)
— the PrivateBin project's own recommended image
([PrivateBin/docker-nginx-fpm-alpine](https://github.com/PrivateBin/docker-nginx-fpm-alpine)).
Pin a release tag (here `2.1.1`).

## Deployment Notes

- **Zero-knowledge, so TLS matters.** The encryption key travels in the URL
  fragment and never hits the server — but you still want the whole thing over
  **HTTPS** so the ciphertext and page assets aren't tampered with in transit.
  Put it behind your reverse proxy + TLS (hardening entry 09).
- **Storage:** default backend is the filesystem, under the `privatebin-data`
  volume — that's your only backup target. (A database backend is possible but
  unnecessary for a personal instance.)
- **Configuration is optional.** Defaults are sensible. To change expiry
  options, size limits, enable discussions, or tweak features, mount your own
  `conf.php` at `/srv/cfg/conf.php` (uncomment in the compose) — base it on the
  project's `conf.sample.php`.
- **Port:** app on container `8080`, published as `${PRIVATEBIN_PORT}` (default
  `8094`). Set `LISTEN_ADDR=127.0.0.1` to keep it host-only behind your proxy.
- **Good hardening candidate.** The image runs as a non-root user on a high
  port, so `cap_drop: [ALL]` fits (shown commented, Level 2 — its entrypoint
  chowns the data dir, so keep `CHOWN`/`SETUID`/`SETGID` or test carefully).

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — LISTEN_ADDR, port, TZ
docker compose up -d
# open http://<host>:8094 → paste text → set expiry/burn-after-reading → share the link
```

> Keep the **full** link including everything after `#` — that fragment *is* the
> decryption key. A link without it can't be decrypted (by anyone, including
> you).

## Thanks

Thanks to the [PrivateBin](https://github.com/PrivateBin/PrivateBin) project for
a genuinely private, well-maintained pastebin.

## Links

- Project: https://privatebin.info/
- Source: https://github.com/PrivateBin/PrivateBin
- Docker image: https://github.com/PrivateBin/docker-nginx-fpm-alpine
- Configuration: https://github.com/PrivateBin/PrivateBin/wiki/Configuration
