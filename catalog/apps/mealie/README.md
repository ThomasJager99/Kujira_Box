This directory contains an example deployment of Mealie, a self-hosted recipe
manager and meal planner. Mealie is a private home for all your recipes: paste a
URL from almost any cooking site and it scrapes the ingredients and steps into a
clean, ad-free recipe card; add your own; then organize everything with
categories, tags and cookbooks. On top of that it does meal planning (drag
recipes onto a calendar), auto-generates shopping lists from what you plan to
cook, and serves it all through a mobile-friendly web UI — so your recipe
collection lives on your own server instead of scattered across bookmarks and
ad-choked websites.


<p align="center">
  <img src="../../../assets/logos/mealie.svg" width="180">
</p>

<br>


## Official container image

https://github.com/mealie-recipes/mealie — image published as `ghcr.io/mealie-recipes/mealie`.

## Deployment Notes

- Runs as a two-container stack: the **Mealie** app and a **PostgreSQL** database
  that holds recipes, plans and metadata.
- Stores recipe images, uploads and backups in the `mealie_data` volume
  (`/app/data`); the database lives in its own volume.
- Startup ordering is enforced with a healthcheck: Mealie waits for PostgreSQL to
  report healthy before booting.
- Exposes its web UI on container port `9000` (published on host `9925` here).
- `BASE_URL` must match the address you actually use, so links and shares are
  correct; `ALLOW_SIGNUP` gates new-account registration (turn it off after setup).
- Healthchecks: PostgreSQL uses `pg_isready`; Mealie uses a small Python probe
  against its `/api/app/about` endpoint (the image has Python but not curl).
- Memory and CPU limits are set on both services.
- Environment-specific values (images, port, URL, DB credentials) are in `.env`.
  Pin `MEALIE_IMAGE` to a specific version for a stable production deployment.
- **Backups:** a `pg_dump` of the database plus the `mealie_data` volume (images
  and uploads) together are a complete restore.

## Example Use

Mealie can be used to:

- import recipes from a URL — it scrapes the page into a clean recipe card
- store and organize your own recipes with categories, tags and cookbooks
- plan meals on a calendar and auto-build shopping lists from the plan
- scale ingredient quantities and follow recipes hands-free on a phone/tablet
- share recipes or keep a private household cookbook
- replace scattered bookmarks and ad-heavy recipe sites with one private library

This gives a household a tidy, searchable, self-hosted cookbook and meal planner.

## Thanks

Thanks to the Mealie project and its maintainers for a polished, genuinely
pleasant, open-source recipe manager.

## Links

- Source: https://github.com/mealie-recipes/mealie
- Documentation: https://docs.mealie.io
