# 18 — Know what's inside: SBOMs with Syft

Entry 17 ended on a cliffhanger: Grype can scan a **SBOM** instead of an image.
This entry is what that means. A SBOM — Software Bill of Materials — is a
complete, machine-readable **ingredient list** of an image: every package, its
version, and its license. **Syft** (by Anchore, like Grype) generates one. Once
you have the ingredient list, scanning, auditing and incident response all get
faster and more honest, because you're no longer guessing what's inside a black
box.

## The risk: you can't secure what you can't see

An image is opaque. You pulled `some/app:1.2`, but what's *actually* in it — which
OpenSSL, which log4j, which curl? Without an inventory you only find out by
re-scanning the whole image every time, and you can't answer the question that
matters most during a security scare:

> "A critical CVE just dropped in library X. Am I running it anywhere?"

With SBOMs that's a text search across files. Without them it's a frantic re-pull
and re-scan of every image you run, hoping you remember them all.

## Syft: generate the ingredient list

```bash
# list everything inside an image (run Syft itself as a container):
docker run --rm anchore/syft:latest privatebin/nginx-fpm-alpine:2.1.1

# save it as a file, in a standard format, to keep and re-use:
docker run --rm anchore/syft:latest \
  privatebin/nginx-fpm-alpine:2.1.1 -o spdx-json > privatebin.sbom.json
```

Output: a table of every OS package and language dependency (name · version ·
type). The `-o` flag writes a standard format: **`spdx-json`** or
**`cyclonedx-json`** are the two industry standards (increasingly *required* —
US executive order, EU Cyber Resilience Act). Syft also reads directories and
archives (`dir:./project`), not just images.

## Why an SBOM beats re-scanning the image

1. **Scan exactly what you shipped.** Generate the SBOM once at build time, then
   feed it to Grype (entry 17) forever after:
   ```bash
   docker run --rm anchore/syft:latest myapp:1.2 -o json > sbom.json
   docker run --rm anchore/grype:latest sbom:sbom.json
   ```
   No re-pulling the image; you're scanning the precise artifact you released.
2. **Instant "am I affected?"** Store SBOMs as build artifacts (a perfect job for
   yesterday's Garage bucket). When the next log4shell lands, `grep` your SBOMs
   instead of re-scanning your whole fleet.
3. **It's the foundation for the next steps.** An SBOM is what you *sign* and
   *attest* to (cosign / SLSA — coming up): "here is proof of exactly what this
   image contained when I built it."

## Make it a habit

- **At build time (CI):** `syft <image> -o cyclonedx-json` → save the SBOM next
  to the image as an artifact. One extra pipeline step.
- **Keep them:** an SBOM per release, archived (Garage). Your searchable history
  of what shipped when.
- **Scan the SBOM, not the image,** on every recurring check — faster and exact.

## How to verify (try it now)

```bash
# ingredient list of today's service image:
docker run --rm anchore/syft:latest ghcr.io/garethgeorge/backrest:v1.14.1

# save an SBOM, then scan THAT with Grype (entry 17) — no image re-read:
docker run --rm anchore/syft:latest ghcr.io/garethgeorge/backrest:v1.14.1 \
  -o json > backrest.sbom.json
docker run --rm -v "$PWD:/w" -w /w anchore/grype:latest --only-fixed sbom:backrest.sbom.json
```

Compare: scanning the SBOM gives the same findings as scanning the image, but
instantly and offline, because the hard part (reading the image) was done once.

## Today's example

Backrest's image "includes rclone and common Unix utilities," so its SBOM is
longer and more interesting than a `FROM scratch` one — a good specimen to
actually *read*. Generate it and skim the list: you'll see the OS base packages,
rclone, and the Go runtime bits. That list is now something you own and can
search — the difference between "I think it's fine" and "I can show you exactly
what's in it."

## Links

- Syft: https://github.com/anchore/syft
- SBOM formats — SPDX: https://spdx.dev/ · CycloneDX: https://cyclonedx.org/
- Syft + Grype together: https://github.com/anchore/grype#using-a-local-sbom
