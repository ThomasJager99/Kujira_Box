# Memos

**Memos** is a lightweight, self-hosted note-taking app with the feel of a
private microblog. You jot short notes in **Markdown** — a thought, a snippet, a
link, a TODO — tag them with `#hashtags`, and they stack up in a clean
chronological timeline you can search and filter. It's the tool for *quick
capture*: not a heavy wiki or document editor, but the place your fleeting ideas
land before they're lost. It has a REST API, public/private memo visibility,
resource (image/file) uploads, and an explore feed, yet runs from a single tiny
container. For a homelab it's the zero-friction "brain dump" app that keeps your
notes on your own server instead of a cloud notes service.


<p align="center">
  <img src="../../../assets/logos/memos.svg" width="180">
</p>

<br>


## Official container image

- **Image:** `neosmemo/memos`
- **Project:** https://github.com/usememos/memos
- **Docs:** https://www.usememos.com/docs

## Deployment Notes

- **One container.** Single small `memos` service (Go, Alpine-based). Defaults to
  an embedded **SQLite** database — plenty for personal use. It can be pointed at
  PostgreSQL/MySQL via `MEMOS_DRIVER` + `MEMOS_DSN` if you want (then it becomes
  a multi-container stack).
- **Persistent state.** Named volume `memos_data` (`/var/opt/memos`) holds the
  SQLite database and uploaded resources. Back it up.
- **Single HTTP interface** on port `5230`.
- **First run.** The very first account you register becomes the **host/admin**
  — so register yourself immediately, before exposing it, and turn off open
  sign-ups in settings if you don't want others joining.
- **Pinned version.** `.env` pins `MEMOS_VERSION` (notes = data → pin and
  upgrade on purpose). Verified latest at build time: `v0.31.0`.
- **Healthcheck** fetches `/healthz` with the image's busybox `wget`.
- **Optional hardening** (commented in compose): `cap_drop: [ALL]`,
  `read_only: true`, `tmpfs: [/tmp]` — enable after testing.
- **Resource limits** and all environment-specific values are externalized to
  `.env`.

## Example Use

- Capture quick thoughts, snippets and links in a searchable timeline.
- Keep a lightweight daily log / journal with `#tags`.
- Save code snippets and command notes with Markdown formatting.
- Use the API to push notes from scripts or share selected memos publicly.

## Thanks

To the **Memos (usememos)** project and its contributors for a fast, private,
no-friction note app.

## Links

- Project: https://github.com/usememos/memos
- Documentation: https://www.usememos.com/docs
- Docker guide: https://www.usememos.com/docs/install/container-deployment
