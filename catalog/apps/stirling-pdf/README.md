This directory contains an example deployment of Stirling-PDF, a self-hosted,
web-based toolkit for working with PDF files — a private alternative to the
countless "free online PDF" sites. It bundles 50+ tools behind one clean web
interface: merge and split, convert to and from images and office documents,
compress, rotate and reorder pages, run OCR to make scans searchable, add or
remove passwords, sign, watermark, extract pages, and much more. The whole point
is privacy: your documents are processed entirely on your own server and are
never uploaded to some stranger's website.


<p align="center">
  <img src="../../../assets/logos/stirling-pdf.svg" width="180">
</p>

<br>


## Official container image

https://github.com/Stirling-Tools/Stirling-PDF — image published as
`stirlingtools/stirling-pdf` (also on `docker.stirlingpdf.com`).

## Deployment Notes

- Runs as a single container (a Spring Boot / Java app).
- Persists state in volumes: `configs` (settings and the app database),
  `tessdata` (OCR language files), plus `logs` and `pipeline` (automation).
- Exposes a single HTTP interface on port `8080`.
- **Image variants:** `:latest` is the standard build; `:latest-fat` adds
  LibreOffice and OCR for office-document conversion and scanning; `:latest-ultra-lite`
  is a stripped-down minimal set. Pick `-fat` if you want conversions and OCR.
- Login is **off by default** (`SECURITY_ENABLELOGIN=false`) — fine for a private
  homelab behind your network; set it to `true` to require accounts.
- Includes a healthcheck against the status endpoint, with a generous
  `start_period` because the Java app takes a little while to boot.
- Memory and CPU limits are set (the `-fat` image with LibreOffice is heavier).
- Environment-specific values (image tag, port, locale, login) are externalized to `.env`.
- **Backups:** the `configs` volume holds settings and the database; the rest is
  regenerable. Documents you process aren't stored — they're downloaded back to you.

## Example Use

Stirling-PDF can be used to:

- merge several PDFs into one, or split one into many
- convert PDFs to/from images and (with `-fat`) Word/Excel/PowerPoint
- compress large PDFs, rotate, reorder, delete or extract pages
- OCR scanned documents so their text becomes searchable
- add or remove passwords, sign, watermark and flatten forms
- do all of it privately, without uploading files to a third-party website

This gives an individual or household a full, self-hosted PDF workshop.

## Thanks

Thanks to the Stirling-Tools team and community for a genuinely useful,
feature-packed, open-source PDF toolkit that keeps document handling private.

## Links

- Source: https://github.com/Stirling-Tools/Stirling-PDF
- Documentation: https://docs.stirlingpdf.com
