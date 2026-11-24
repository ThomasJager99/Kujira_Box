# 16 — Docker Scout: the scanner that's already on your machine

Entry 16 used Trivy to find CVEs in an image. Docker Scout does the same job,
but it's **built into the Docker CLI** — no separate tool to pull, and it talks
in terms of your *own* images and a recommended **base-image upgrade** path.
Running two scanners isn't redundant: they use different databases and surface
slightly different findings. Scout is the one that costs you nothing to adopt
because it's already there.

## The risk: same as entry 16 — this is a second opinion

A scanner is only as good as its vulnerability feed, and no feed is complete.
Trivy and Scout draw from overlapping-but-different sources, so one will
occasionally flag a fixable Critical the other misses. More importantly, Scout
is *frictionless*: `docker scout quickview` is one command against an image you
already built or pulled, which makes it the scan you'll actually run as a habit.

## Quickview: the 10-second triage

```bash
# summary table: how many CVEs by severity, in the image and its base
docker scout quickview privatebin/nginx-fpm-alpine:2.1.1
```

`quickview` prints a compact grid — your image vs. its base image, counts of
Critical/High/Medium/Low. It's the "should I worry?" glance before you dig in.

## cves: the detailed list

```bash
# full CVE list, only fixable High/Critical (the actionable ones)
docker scout cves --only-severity critical,high --only-fixed \
  privatebin/nginx-fpm-alpine:2.1.1
```

- `--only-fixed` = hide CVEs with no patch available (same idea as Trivy's
  `--ignore-unfixed`): don't stare at things you can't action.
- `--only-severity critical,high` = cut the Low/Medium noise.
- Each entry names the package, the CVE, and the version that fixes it.

## recommendations: Scout's party trick

```bash
docker scout recommendations privatebin/nginx-fpm-alpine:2.1.1
```

This is what Scout does that a plain scanner doesn't: it looks at the **base
image** and tells you concrete upgrades — "switch to this newer base tag, clear
N CVEs" or "this slimmer variant removes M packages". It turns "you have
vulnerabilities" into "change this one line in your Dockerfile". For a homelab
where most of your risk is stale base images, that's the high-value output.

## Where it runs

`docker scout` ships with recent Docker Desktop and Docker Engine. Check:

```bash
docker scout version   # if missing, it's a CLI plugin you can install once
```

The vulnerability data is fetched from Docker's servers, so the first run needs
network. Scanning a **local** image you just built works the same way — handy
for checking your own `docker build` output before you ship it.

## Scout vs. Trivy — which, when

| | Docker Scout | Trivy (entry 16) |
|---|---|---|
| Install | already in Docker | run as a container |
| Best feature | base-image **recommendations** | breadth: `fs`, `config`, secrets |
| Habit fit | `quickview` is one glance | great in CI / scripting |
| Data source | Docker's advisory DB | Aqua's + multiple feeds |

**Use both, lightly.** Scout `quickview` for the daily "is this image OK"
glance and its upgrade advice; Trivy when you want to scan a repo, a compose
file, or wire a gate into CI. Agreement between them is reassuring; a finding in
only one is still worth a look.

## Make it a habit

- **`docker scout quickview <image>`** right after you pull or build something
  new — before `docker compose up`.
- **`docker scout recommendations <image>`** when quickview looks bad — it
  usually hands you a one-line base-image fix.
- Re-run occasionally on your running stack; new CVEs land against images that
  never changed.

## How to verify (try it now)

```bash
# today's service image — a FROM-scratch static binary, almost nothing to scan:
docker scout quickview dxflrs/garage:v2.6.0

# a fatter image for contrast — see what packages + base actually look like:
docker scout quickview postgres:16
docker scout recommendations postgres:16   # Scout's one-line base upgrade advice

# compare Scout vs Trivy on the same image — note where the two disagree:
docker run --rm aquasec/trivy:latest image --severity HIGH,CRITICAL \
  --ignore-unfixed postgres:16
```

Seeing the two tools' numbers side by side is the lesson: no single scanner is
the whole truth.

## Today's example

Today's service (Garage) ships `FROM scratch` — just a static binary, no OS
packages at all — so a scanner finds **almost nothing**. That's the whole point
of a minimal image: no shell, no libc, no package manager means almost no CVEs
and almost no attack surface. Run `quickview` on it, then on a `postgres:16` or a
Node image, and the contrast *is* the lesson — the slimmer the base, the less a
scanner (and an attacker) has to work with. Scout's `recommendations` exists to
nudge the fat images down toward that.

## Links

- Docker Scout: https://docs.docker.com/scout/
- `docker scout` CLI reference: https://docs.docker.com/reference/cli/docker/scout/
- Scout CLI (standalone plugin): https://github.com/docker/scout-cli
- Compared with Trivy: see entry 16
