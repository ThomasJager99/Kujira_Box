# SearXNG

**SearXNG** is a self-hosted **metasearch** engine: you type one query and it
fetches results from dozens of other engines (Google, Bing, DuckDuckGo,
Wikipedia, GitHub, and 200+ more) in parallel, then aggregates and presents them
— **without tracking you, profiling you, or serving ads**. Because the searches
go out from *your* server and SearXNG strips identifying data, the upstream
engines can't build a profile on you, and there's no history stored about what
you searched. You can pick exactly which engines to use, search by category
(web, images, news, code, maps, science), and even set it as your browser's
default search engine. For a homelab it's the "take back your search privacy"
tool.


<p align="center">
  <img src="../../../assets/logos/searxng.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `searxng/searxng`
- **Project:** https://github.com/searxng/searxng
- **Docs:** https://docs.searxng.org/

## Deployment Notes

- **One container.** Single `searxng` service (Python, Alpine-based). For a
  busy/public instance the vendor also recommends a Redis/Valkey sidecar for
  rate-limiting; for a personal instance this single container is fine.
- **Single HTTP interface** on port `8080`.
- **Persistent state.** Named volume `searxng_config` (`/etc/searxng`) holds
  `settings.yml`. On first start the image creates it; **set a strong
  `server.secret_key`** there (the image generates one, but confirm it's not the
  placeholder) and enable the `limiter` / extra result formats (e.g. `json`) as
  you like.
- **Base URL.** `SEARXNG_BASE_URL` must match how you actually reach it, or
  generated links and some engines misbehave.
- **Built-in hardening (live example).** This stack ships the vendor-recommended
  least-privilege set: `cap_drop: [ALL]` then `cap_add: [CHOWN, SETGID, SETUID]`
  — only the three capabilities the entrypoint needs to drop to its unprivileged
  user and fix permissions — plus `no-new-privileges`. This is a concrete,
  known-good example of the hardening the other stacks leave commented.
- **Pinned version.** SearXNG uses rolling date/commit tags; pin one from the
  registry for reproducible rebuilds.
- **Healthcheck** fetches `/healthz` with the image's busybox `wget`.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Search the web privately — no ads, no tracking, no query history.
- Aggregate results from many engines at once and compare them.
- Set it as your browser's default search engine (it exposes an OpenSearch
  descriptor).
- Search niche categories: code, maps, science papers, images, news.

## Thanks

To the **SearXNG** project and its contributors for a private, open,
ad-free way to search the web.

## Links

- Project: https://github.com/searxng/searxng
- Documentation: https://docs.searxng.org/
- Admin / settings: https://docs.searxng.org/admin/installation-searxng.html
