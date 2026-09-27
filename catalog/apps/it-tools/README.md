This directory contains an example deployment of IT-Tools, a self-hosted
collection of around a hundred small, handy utilities for developers and IT
people, all in one clean web interface. Instead of googling a random "online
JSON formatter" or "base64 decoder" (and pasting your data into who-knows-whose
website), you host the whole toolbox yourself: token and password generators,
hash and HMAC, base64 / URL / JWT encode-decode, JSON/YAML/SQL formatters, cron
expression parser, UUID and ULID generators, color converters, QR codes, regex
tester, and many more. Everything runs **client-side in your browser**, so the
data you paste never leaves your machine.


<p align="center">
  <img src="../../../assets/logos/it-tools-light.svg" width="180">
</p>

<br>


## Official container image

https://github.com/CorentinTh/it-tools — image published as `ghcr.io/corentinth/it-tools`.

## Deployment Notes

- Runs as a single, **stateless** container — it's a static site served by nginx,
  so there is no database, no volumes and no persistent data to manage.
- All tools run **in the browser** (client-side); pasted data never reaches the
  server, which is the whole privacy point.
- Exposes the web interface on container port `80` (published on host `8080` here).
- Includes a lightweight healthcheck against the web root.
- Memory and CPU limits are minimal — it barely uses anything.
- Environment-specific values (image tag, port) are externalized to `.env`.
- **Backups:** none needed — nothing is stored. Just redeploy the image.

## Example Use

IT-Tools can be used to:

- generate passwords, tokens, UUIDs/ULIDs and hashes
- encode/decode base64, URL, JWT, and format JSON / YAML / SQL / XML
- parse and explain cron expressions, test regular expressions
- convert colors, generate QR codes, do date/number/base conversions
- keep a private, ad-free "developer swiss-army knife" on your own server

This gives you one self-hosted toolbox instead of scattering your data across
dozens of random online utility sites.

## Thanks

Thanks to Corentin Thomasset and the IT-Tools contributors for bundling a huge
set of everyday utilities into one clean, open-source, self-hostable app.

## Links

- Source: https://github.com/CorentinTh/it-tools
- Hosted demo: https://it-tools.tech
