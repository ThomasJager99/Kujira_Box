# 15 — Image provenance: do you trust this image?

This is the question the whole security thread started from: *"everything might
contain malicious code — how do I check?"* Block B is the answer, and it begins
here, at the first decision you make about any new service: **before you pull an
image, should you trust it at all?** Provenance — who made it, from what, and
how — is how you decide.

## The risk: `docker pull` runs someone else's code on your box

Pulling an image and running it is running a stranger's software with whatever
access your compose file grants it. A malicious or compromised image doesn't
need to escape the container to hurt you (entry 01) — it can exfiltrate the data
you mount and the secrets you pass, and phone home, all from *inside*. The
supply chain has real attacks:

- **Typosquatting** — `postgress`, `redis-alpine`, `nginx-proxy` — names one
  character or one hyphen off a popular image, hoping you fat-finger it.
- **Abandoned/hijacked images** — a popular community image whose maintainer
  account got compromised, or who quietly handed it off.
- **Backdoored "convenience" images** — a repackaged "easy" version of a popular
  app that adds a miner or a data-stealer.
- **Stale images** — not malicious, just years-old with known-exploitable CVEs
  (the scanning entries 16–18 cover these).

So before the first `docker pull`, you vet the source.

## The provenance checklist

### 1. Who publishes it?

Prefer, in rough order of trust:

- **Docker Official Images** — the curated `_/library` images (`postgres`,
  `redis`, `nginx`, `mariadb`). Reviewed, maintained, the gold standard. On
  Docker Hub they show an **"Official image"** badge and have **no namespace**
  (`postgres`, not `someuser/postgres`).
- **Verified Publisher / the project's own org** — the vendor's official account
  (`grafana/grafana`, `nextcloud/`, `ghcr.io/home-assistant/...`), or a trusted
  umbrella like **LinuxServer.io** (`lscr.io/linuxserver/*`). Look for the
  Verified Publisher badge, or that it's the GitHub org that owns the source.
- **A real project with a real repo behind it** — a named maintainer, an active
  GitHub, issues being answered, regular releases.

Be wary of: random one-off Docker Hub accounts, images with no linked source,
no README, tiny pull counts on a supposedly popular tool, or "fixed/better"
forks of official images.

### 2. Does a real source repo stand behind it?

The image should trace back to **source you can read**. Open the GitHub repo
linked from Docker Hub:

- Does the repo actually build *this* image (a visible `Dockerfile` / CI that
  pushes to this tag)?
- Is it active — recent commits, releases matching the image tags, maintainers
  responding?
- Stars/forks/issues consistent with a real, used project?

"No source linked" is a red flag on its own. You can't vet what you can't see.

### 3. Read the Dockerfile

For anything you're unsure about, actually read how it's built. You're looking
for:

- **`curl ... | bash` / `wget ... | sh`** pulling remote scripts at build or
  run time — unpinned remote code execution baked into the image.
- **Base image**: is it `FROM` something sane (`alpine`, `debian-slim`, an
  official runtime) or `FROM some/random-image`?
- **Secrets or odd binaries** baked in, weird `ENV`, calls to unexpected hosts.
- **Pinned dependencies** vs. "install latest of everything from the internet."

### 4. Check it resolves & matches

```bash
# inspect the manifest WITHOUT running anything:
docker buildx imagetools inspect postgres:16-alpine
# -> shows the real digest, platforms, and (if present) attestations/SBOM

# how big, how built, what layers (a tiny "full app" image is suspicious):
docker manifest inspect <image>
```

## This series' own rule

Every service in this catalog deliberately uses **official or
verified-publisher images** — `postgres`, `mariadb`, `redis`,
`ghcr.io/<project>/...`, `lscr.io/linuxserver/...`, or the maintainer's own org
(`snipe/snipe-it`, `metabase/metabase`, `infisical/infisical`). That's not an
accident; it's provenance as a default. When a tool only had a sketchy image, it
didn't make the catalog. (This is also why "Vyrakrypt" — a name with **no repo,
no image, no docs** — got rejected on sight: zero provenance is an instant no.)

## How to verify (quick gut-check before pulling)

```bash
# 1. Is it an official or namespaced-by-the-project image? (read the name)
#    postgres            -> official
#    grafana/grafana     -> project org
#    lscr.io/linuxserver -> trusted umbrella
#    randojoe/postgres   -> ??? vet hard or avoid

# 2. Does Docker Hub / GHCR link to a real, active source repo? (open it)

# 3. Pull counts + last-updated sane for a "popular" tool?

# 4. For anything unsure: read its Dockerfile in that repo.
```

If you can't answer "who made this and from what," don't run it — or run it in a
throwaway VM first (block C).

## Today's example

Snipe-IT (today's service) passes cleanly: the image is **`snipe/snipe-it`** —
the maintainer's own namespace, backed by the active
[snipe/snipe-it](https://github.com/snipe/snipe-it) GitHub repo with matching
release tags — and its database is the **official `mariadb`**. Known publisher,
visible source, versioned tags: that's what "good provenance" looks like, and
it's the five-second check to run on *every* new image before it touches your
host.

## Links

- Docker Official Images: https://docs.docker.com/trusted-content/official-images/
- Verified Publisher: https://docs.docker.com/trusted-content/dvp-program/
- `docker buildx imagetools inspect`: https://docs.docker.com/reference/cli/docker/buildx/imagetools/inspect/
