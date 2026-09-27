This directory contains an example deployment of Gitea, a lightweight, open-source, self-hosted Git service — essentially your own private GitHub or GitLab. Gitea gives you a fast web interface for hosting Git repositories with the full workflow around them: issues, pull requests, code review, wikis, releases and a built-in package registry. It even ships Gitea Actions, a CI/CD system compatible with GitHub Actions workflow syntax, so you can run pipelines on your own runners. Written in Go, it's tiny and quick even on modest hardware, and everything — code, history, issues — lives on a server you control.


<p align="center">
  <img src="../../../assets/logos/gitea.svg" width="180">
</p>

<br>


## Official container image

https://github.com/go-gitea/gitea — image published as `gitea/gitea`.

## Deployment Notes

- Runs as a two-container stack: the **Gitea** server and a **PostgreSQL** database that holds all metadata (users, repos, issues, pull requests).
- Stores state in persistent Docker volumes: `gitea_data` (repositories, config, avatars, LFS) and `gitea_pgdata` (the database).
- Exposes two ports: HTTP for the web UI/API (container `3000`) and SSH for `git` over SSH (container `22`, published on host `222` by default so it doesn't clash with the host's own sshd).
- Startup ordering is enforced with a healthcheck: the Gitea server only starts once PostgreSQL reports healthy, so first-run setup doesn't race an unready database.
- Configured entirely through `GITEA__section__KEY` environment variables (database and public address), so no manual editing of `app.ini` is needed to get started.
- `GITEA_DOMAIN` / `GITEA_ROOT_URL` / `GITEA_SSH_PORT` are set so clone URLs and links Gitea generates are correct — point them at your real host/domain.
- Memory and CPU limits are set on both services to keep the stack polite on a shared homelab host.
- Environment-specific values (image tags, ports, public address, database credentials) are externalized to a `.env` file.
- **Backups:** dump the PostgreSQL database (`pg_dump`) and back up the `gitea_data` volume (it holds the actual repositories). Those two together are a complete restore.

## Example Use

Gitea can be used to:

- host private Git repositories with issues, pull requests and code review
- run CI/CD pipelines with Gitea Actions (GitHub Actions-compatible syntax)
- mirror repositories to/from GitHub or other remotes
- publish packages (container images, npm, PyPI, and more) via the built-in registry
- organize work across teams and organizations with fine-grained access control
- keep a fully private, self-hosted alternative to GitHub/GitLab

This gives an individual or team their own Git platform — from code hosting to CI — without depending on a third-party service.

## Thanks

Thanks to the Gitea project and its maintainers for a fast, lightweight, feature-complete Git service that makes self-hosting your own code platform genuinely easy.

## Links

- Source: https://github.com/go-gitea/gitea
- Website: https://about.gitea.com
- Documentation: https://docs.gitea.com
