# 16 — Scanning images for CVEs with Docker Scout

Entry 15 checked *who made* an image — that it comes from a publisher you trust
and isn't malicious. This entry checks *what's inside* it. Even an image from a
publisher you trust can ship old libraries with known, published
vulnerabilities — because it was built months ago, or its base image drifted. A
scanner reads the image and tells you exactly which known CVEs it contains.
**Docker Scout** is the one to start with, because it's already built into the
Docker CLI — nothing extra to install.

## The risk: trusted ≠ patched

Provenance (entry 15) tells you an image isn't *malicious*. It doesn't tell you
it isn't *vulnerable*. Every image is a frozen snapshot of an OS + libraries at
build time, and vulnerabilities are discovered continuously. So:

- An official image pinned six months ago may contain a dozen CVEs disclosed
  *since* it was built.
- A `:latest` you pulled once and never updated silently rots.
- A base image (`node`, `python`, `debian`) carries OS packages with their own
  CVE stream, independent of the app.

You can't eyeball this — CVEs live in databases, matched against exact package
versions. That's what a scanner does, and Docker already ships one.

## Scout is already on your machine

`docker scout` comes with recent Docker Desktop / Docker Engine — no separate
tool to pull. Check it's there:

```bash
docker scout version   # if missing, it's a one-time CLI-plugin install
```

The vulnerability data is fetched from Docker's servers, so the first run needs
network. It scans both local images you just built and anything you've pulled.

## quickview — the 10-second triage

```bash
docker scout quickview postgres:16-alpine
```

`quickview` prints a compact grid — your image vs. its base image, counts of
Critical/High/Medium/Low. It's the "should I worry?" glance before you dig in.

## cves — the detailed list

```bash
# full CVE list, only fixable High/Critical (the actionable ones)
docker scout cves --only-severity critical,high --only-fixed postgres:16-alpine
```

- `--only-fixed` = hide CVEs with no patch available: you can't action those
  anyway, so they're just noise. Focus on what you *can* patch.
- `--only-severity critical,high` = cut the Low/Medium noise.
- Each entry names the package, the CVE, and **the version that fixes it** —
  that last part is the gold: it tells you whether an update actually exists.

### Reading the results sanely

A long list isn't a crisis — triage it. A **fixable** High/Critical means: update
the image. Many findings vanish just by moving from a months-old pin to the
current release, or from a `debian` base to a `-slim`/`-alpine` one. Some CVEs
are in components your app never calls — real but low-risk. Perfect patching
isn't the goal; **knowing, and clearing the easy wins, is.**

## recommendations — Scout's party trick

```bash
docker scout recommendations postgres:16-alpine
```

This is what Scout does that a plain scanner doesn't: it looks at the **base
image** and hands you concrete upgrades — "switch to this newer base tag, clear
N CVEs" or "this slimmer variant removes M packages". It turns "you have
vulnerabilities" into "change this one line in your Dockerfile". For a homelab
where most of your risk is stale base images, that's the high-value output.

## One scanner is never the whole truth

No scanner's vulnerability feed is complete. Scout is the frictionless default —
but for anything important it's worth a **second opinion** from another scanner
that draws on different databases, like **Grype** (next entry) or **Trivy**.
Agreement between two is reassuring; a finding in only one is still worth a look.
Scout's edge is the `recommendations` command; a tool like Trivy's is breadth —
it can also scan a filesystem, a git repo, or a compose file for
misconfigurations. Use the built-in one daily, reach for a second when it counts.

## Make it a habit, not a one-off

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
```

First run downloads the vulnerability data (a few seconds); later runs are fast.

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
- A second scanner (Grype) follows in the next entry.
