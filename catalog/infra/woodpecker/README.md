# Woodpecker CI

Woodpecker is a **lightweight, self-hosted CI/CD engine** — the open-source
spiritual successor to Drone. It plugs into your Git forge (Gitea, Forgejo,
GitHub, GitLab…), watches your repos, and runs the pipeline described in a
`.woodpecker.yml` file on every push or pull request. Each step runs in its own
container, so your builds are clean and reproducible. It's a natural companion
to a self-hosted **Gitea** — your code and your CI, both on your own box.

It comes in two pieces:

- **server** — the web UI, the brain: authenticates against your forge, stores
  pipeline state (SQLite by default), and hands work out to agents.
- **agent** — the muscle: connects back to the server and actually executes
  each pipeline step as a Docker container.



<p align="center">
  <img src="../../../assets/logos/woodpecker-ci.svg" width="180">
</p>

<br>


## Official container images

- Server: [`woodpeckerci/woodpecker-server`](https://hub.docker.com/r/woodpeckerci/woodpecker-server)
- Agent: [`woodpeckerci/woodpecker-agent`](https://hub.docker.com/r/woodpeckerci/woodpecker-agent)

Project: [woodpecker-ci/woodpecker](https://github.com/woodpecker-ci/woodpecker).
Pin both to the **same** release tag (here: `v3.20.1`) — a mismatched
server/agent pair can refuse to talk.

## Deployment Notes

- **Connect your forge first.** In Gitea: *Settings → Applications → Create a
  new OAuth2 application*, with redirect URI `${WOODPECKER_HOST}/authorize`.
  Paste the generated client id + secret into `.env`
  (`WOODPECKER_GITEA_CLIENT` / `WOODPECKER_GITEA_SECRET`).
- **`WOODPECKER_HOST`** must be the exact public URL you reach the server on —
  it's used for the OAuth round-trip and webhooks. Behind a reverse proxy,
  that's your `https://ci.example.com`, not `http://host:8000`.
- **`WOODPECKER_AGENT_SECRET`** is a shared secret — generate it once
  (`openssl rand -hex 32`) and use the identical value for server **and** agent.
- **`WOODPECKER_OPEN=false`** keeps the instance closed: only admins listed in
  `WOODPECKER_ADMIN` can enable repositories. Leave it closed unless you want
  every forge user to onboard their own repos.
- **Ports:** `8000` (web UI) and `9000` (gRPC, how agents connect). Only the web
  port needs to reach users — put it behind your reverse proxy with TLS.

### ⚠️ Security: the agent is privileged by nature

The agent mounts `/var/run/docker.sock` because its entire job is to **start
containers** for your pipeline steps. Mounting the Docker socket is equivalent
to giving that container **root on its host** — and a CI agent runs *whatever
code is in the repo it builds*. That's an unavoidable combination, so treat it
with respect:

- Run the agent on a **CI-only / disposable host**, not alongside your sensitive
  services (ideally a dedicated VM — see the hardening rubric's sandbox block).
- A `docker-socket-proxy` doesn't really help here: the agent needs to *create*
  containers, which is the dangerous capability, not a read-only one.
- Only build repositories you trust, and keep the instance closed (`OPEN=false`).

This stack is the real-world example behind the hardening entry on the Docker
socket — see [`hardening/`](../../hardening/).

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — WOODPECKER_HOST, the Gitea OAuth client/secret, AGENT_SECRET, ADMIN
docker compose up -d
# open the web UI → "Login with Gitea" → authorize → enable a repo
```

Then drop a `.woodpecker.yml` in that repo:

```yaml
steps:
  - name: test
    image: golang:1.23
    commands:
      - go test ./...
```

Push, and Woodpecker runs it.

## Thanks

Thanks to the [Woodpecker CI](https://github.com/woodpecker-ci/woodpecker)
project and its community for a genuinely lightweight, forge-native CI.

## Links

- Docs: https://woodpecker-ci.org/docs/intro
- Source: https://github.com/woodpecker-ci/woodpecker
- Pipeline syntax: https://woodpecker-ci.org/docs/usage/workflow-syntax
