# Garage

Garage is a **lightweight, self-hosted, S3-compatible object storage** server by
[Deuxfleurs](https://garagehq.deuxfleurs.fr/). It speaks the Amazon S3 API, so
anything that can talk to S3 can use it: **restic/rclone backups**, CI build
artifacts, app storage (Nextcloud, Immich, Mastodon…), and — handy for your
IaC work — a **remote-state backend for Terraform**. It's a single static Rust
binary, runs happily on one node, and scales to a geo-distributed cluster if you
ever want redundancy.

Garage is **headless** — there is no web dashboard. You administer it with the
`garage` CLI and use any S3 client (aws-cli, `mc`, rclone, s3cmd) for the data.


## Official container image

[`dxflrs/garage`](https://hub.docker.com/r/dxflrs/garage) — the project's own
image, built `FROM scratch` (just the static binary, no shell or OS). Pin a
release tag (here `v2.6.0`).

> **Why Garage and not MinIO?** MinIO used to be the default pick here, but in
> 2025 the open-source build dropped its web console and moved LDAP/OIDC into a
> paid product, and the `minio/minio` repo was **archived (read-only) in April
> 2026**. For a *maintained*, drama-free, self-hostable S3, Garage is the clean
> choice today. (Checking whether a project is still alive before you build on
> it is exactly the supply-chain discipline from the hardening series.)

## Deployment Notes

- **Two secrets live in `garage.toml`, not `.env`.** Generate them before first
  start and lock the file down:
  ```bash
  cp garage.toml.example garage.toml
  sed -i "s|REPLACE_ME__openssl_rand_hex_32|$(openssl rand -hex 32)|" garage.toml
  sed -i "s|REPLACE_ME__openssl_rand_base64_32|$(openssl rand -base64 32)|" garage.toml
  chmod 600 garage.toml      # it holds the rpc_secret + admin_token
  ```
- **Back up both volumes.** `garage-meta` is the database (small but critical);
  `garage-data` is the objects themselves. On a single node there's **one copy**
  (`replication_factor = 1`) — Garage is not a backup by itself, so back these up.
- **Admin port is host-only.** Port `3903` (health + Prometheus metrics) is
  pinned to `127.0.0.1` in the compose — don't expose it. Only the S3 API
  (`3900`) follows `LISTEN_ADDR`.
- **No shell in the image.** Run CLI commands against the binary directly:
  `docker compose exec garage /garage <cmd>` (not `/bin/sh`).
- **Great hardening target.** A scratch static binary + all writes in named
  volumes means `read_only: true` + `cap_drop: [ALL]` fit cleanly (shown
  commented, Level 2).

## One-time bootstrap

After `docker compose up -d`, Garage is running but has no storage layout, bucket
or keys yet. Set that up once:

```bash
# 1. See the node and copy the first part of its ID:
docker compose exec garage /garage status

# 2. Give this node a zone + capacity, then apply the layout:
docker compose exec garage /garage layout assign -z home -c 10G <node-id-prefix>
docker compose exec garage /garage layout apply --version 1

# 3. Create a bucket and an access key:
docker compose exec garage /garage bucket create backups
docker compose exec garage /garage key create backups-key
#    ^ prints a Key ID (GK...) and a Secret key — save them, like AWS creds

# 4. Grant the key access to the bucket:
docker compose exec garage /garage bucket allow --read --write --owner backups --key backups-key
```

## Example Use

Point any S3 client at `http://<host>:3900`, region `garage`, **path-style**
addressing (no wildcard DNS needed):

```bash
export AWS_ACCESS_KEY_ID=GK...        # from `key create`
export AWS_SECRET_ACCESS_KEY=...      # from `key create`
aws --endpoint-url http://<host>:3900 --region garage \
    s3 ls
aws --endpoint-url http://<host>:3900 --region garage \
    s3 cp ./file.txt s3://backups/
```

**Terraform remote state** (ties to your IaC goal) — in a `backend "s3"` block
set `endpoints = { s3 = "http://<host>:3900" }`, `region = "garage"`,
`use_path_style = true`, and skip the AWS-specific checks
(`skip_credentials_validation`, `skip_region_validation`, `skip_requesting_account_id`).

## Thanks

Thanks to [Deuxfleurs](https://garagehq.deuxfleurs.fr/) for a genuinely free,
maintained, self-hostable S3 that respects small deployments.

## Links

- Project: https://garagehq.deuxfleurs.fr/
- Source: https://git.deuxfleurs.fr/Deuxfleurs/garage · https://github.com/deuxfleurs-org/garage
- Quick start: https://garagehq.deuxfleurs.fr/documentation/quick-start/
- Config reference: https://garagehq.deuxfleurs.fr/documentation/reference-manual/configuration/
