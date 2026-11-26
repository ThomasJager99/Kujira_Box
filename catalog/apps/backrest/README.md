# Backrest

Backrest is a **web UI for [restic](https://restic.net/)** — the excellent,
encrypted, deduplicating backup tool. Restic is brilliant but command-line only;
Backrest puts a browser front-end on it: define backup **plans** (what, where,
when), run them on a schedule, apply retention/pruning policies, and **browse and
restore** individual files from any snapshot — all without memorising restic
commands. It wraps restic, it doesn't replace it, so your backups stay in the
standard restic format (restorable with plain `restic` if Backrest ever goes
away).

Pairs naturally with **Garage** (an S3 repo you host yourself): back up your
homelab to your own object storage, encrypted, with one tool driving it.


<p align="center">
  <img src="../../../assets/logos/backrest.svg" width="180">
</p>

<br>


## Official container image

[`ghcr.io/garethgeorge/backrest`](https://github.com/garethgeorge/backrest/pkgs/container/backrest)
— the project's own image (also on Docker Hub as `garethgeorge/backrest`). Pin a
release tag (here `v1.14.1`). A minimal `:scratch` variant exists too.

## Deployment Notes

- **The repo password is everything.** Restic encrypts every backup. You set the
  repository password in the UI on first run — **write it down somewhere safe
  (your password manager).** Lose it and the backups are unrecoverable by
  design. This is a feature, not a bug.
- **Back up source is mounted read-only.** `BACKUP_SOURCE` → `/userdata:ro`.
  Backrest only reads it, so even a compromise of the container can't modify your
  live data through that mount.
- **Runs as root** so it can read *every* file in the source. Dropping to a
  non-root user risks silently skipping unreadable files — usually wrong for a
  backup (see the commented hardening note in the compose).
- **Back up the `backrest-config` volume too.** It holds your plans and repo
  definitions. The actual backup data lives in the restic repo (local path, or
  an S3/Garage/B2/SFTP backend you configure), not here.
- **S3 / Garage backend:** Backrest supports all restic backends. For a Garage
  repo, use a repo URI like `s3:http://<garage-host>:3900/<bucket>` with the
  access key + secret from `garage key create`, and set the restic password.
- **Keep the UI private.** It can trigger restores — put it behind your reverse
  proxy (`LISTEN_ADDR=127.0.0.1`), not open on the LAN.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — set BACKUP_SOURCE to the host dir you want backed up
docker compose up -d
# open http://<host>:9898
#   → set an instance password (UI login)
#   → add a Repo (local path /repos/... or an S3/Garage URI) + repo password
#   → add a Plan: path /userdata, schedule, retention → run it
```

Test a **restore** right away — a backup you haven't restored from is just a
hopeful guess.

## Thanks

Thanks to [garethgeorge](https://github.com/garethgeorge) for Backrest, and to
the restic project for the engine underneath.

## Links

- Project / docs: https://github.com/garethgeorge/backrest
- restic (the engine): https://restic.net/
- restic backends (S3, B2, SFTP…): https://restic.readthedocs.io/en/stable/030_preparing_a_new_repo.html
