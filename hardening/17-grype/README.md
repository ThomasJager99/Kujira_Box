# 17 — A second opinion with Grype

Entry 16 scanned images with Docker Scout — the one that's already built into
the Docker CLI. This entry is the follow-through on that entry's closing point:
**no single scanner's vulnerability feed is complete.** For anything you actually
care about, run a *second* scanner that draws on a different database and see
where the two disagree. **Grype** (by Anchore) is the natural second opinion:
fast, scriptable, offline-friendly, and built to live in CI.

## The risk: one feed has blind spots

A scanner is only as good as the vulnerability data behind it. Scout uses
Docker's advisory database; Grype uses Anchore's, which aggregates its own set of
sources. They overlap heavily — but not perfectly. So a fixable Critical that one
tool knows about, the other may not have ingested yet. Relying on a single
scanner means quietly trusting one feed's blind spots. Two independent scanners
turn "it looks clean" into "two different databases both say it's clean," which
is a much stronger statement.

## Grype: fast, scriptable, CI-ready

[Grype](https://github.com/anchore/grype) (image `anchore/grype`) needs no
account and no live connection to a vendor's servers at scan time — it downloads
Anchore's vuln DB once, then scans locally, which makes it ideal for pipelines
and air-gapped boxes.

```bash
# scan an image (runs Grype itself as a container, nothing to install):
docker run --rm anchore/grype:latest privatebin/nginx-fpm-alpine:2.1.1

# only show fixable High/Critical — the actionable rows:
docker run --rm anchore/grype:latest \
  --only-fixed --fail-on high privatebin/nginx-fpm-alpine:2.1.1
```

Typical output: a table of package · installed version · **fix version** · CVE ·
severity. Same shape you learned to read with Scout — installed-vs-fix is still
the column that tells you whether an update exists.

### The two switches that matter

- **`--only-fixed`** — hide CVEs with no available patch (you can't action them
  anyway). Same idea as Scout's `--only-fixed`.
- **`--fail-on high`** — makes Grype **exit non-zero** if anything at that
  severity or above is found. This is the CI switch: drop it into your
  Woodpecker/Gitea pipeline and a build with a new fixable High fails the job.

## Grype's two edges over Scout

1. **CI gating.** `--fail-on` + a clean exit code make it trivial to block a
   merge or deploy on new vulnerabilities. That's the supply-chain control the
   whole B-block builds toward: not just *seeing* CVEs, but *refusing to ship*
   them.
2. **SBOM-aware.** Grype can scan a Software Bill of Materials (from Anchore's
   Syft — the next entry's topic) instead of re-reading the image:
   ```bash
   docker run --rm anchore/grype:latest sbom:./sbom.json
   ```
   Scan the SBOM once in CI, feed it to Grype on every later check — faster, and
   it means you're scanning *exactly* what you shipped.

Grype also scans directories, archives and filesystems (`dir:`, `./path`), not
just images — handy for checking a project's vendored dependencies.

## Make it a habit

- **New image:** scan with Grype *and* Scout once. Agreement is reassuring; a
  finding in only one is still worth a look.
- **In CI:** `grype --fail-on critical <image>` as a gate. Start at `critical`
  so you're not blocked by noise, tighten to `high` once the stack is clean.
- **Periodically:** re-scan running images — new CVEs land against images that
  never changed, and Grype's DB updates catch them.

## How to verify (try it now)

```bash
# scan today's service image with Grype:
docker run --rm anchore/grype:latest --only-fixed privatebin/nginx-fpm-alpine:2.1.1

# now scan the SAME image with Scout (entry 16) and compare the lists:
docker scout cves --only-severity critical,high --only-fixed \
  privatebin/nginx-fpm-alpine:2.1.1

# a deliberately stale base to SEE a real list (scan only — don't run it):
docker run --rm anchore/grype:latest --only-fixed node:18-bullseye
```

Put the Grype and Scout outputs side by side. Where they match, you're confident.
Where they differ, you've just learned why one scanner is never enough.

## Today's example

PrivateBin (today's service) is an **alpine**-based image, so both scanners
should return a short list — Alpine's tiny package surface again. That's the
ideal case to practice the *comparison itself* on: with few findings, it's easy
to line up Grype's rows against Scout's and actually see the small differences
between two vulnerability feeds, instead of drowning in a huge report. Small
image, clear lesson.

## Links

- Grype: https://github.com/anchore/grype
- Install / Docker usage: https://oss.anchore.com/docs/installation/grype/
- `--fail-on` and CI usage: https://github.com/anchore/grype#continuous-integration
- Syft (SBOMs, the next entry): https://github.com/anchore/syft
