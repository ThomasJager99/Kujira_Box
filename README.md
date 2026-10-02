# Kujira Box 🐋

> Homelab in a box. — **v0.1.0** (early, actively developed)

[![ShellCheck](https://github.com/ThomasJager99/Kujira_Box/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/ThomasJager99/Kujira_Box/actions/workflows/shellcheck.yml)
[![Python Lint](https://github.com/ThomasJager99/Kujira_Box/actions/workflows/python-lint.yml/badge.svg)](https://github.com/ThomasJager99/Kujira_Box/actions/workflows/python-lint.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

Welcome 👋

Kujira Box is step zero of a much bigger idea: making it easy for anyone —
not just sysadmins — to run useful software on hardware they own.

## The vision (north star)

The long-term goal is a full **application, with an API and AI assistants**, that:

- takes what you *want* (your needs, your budget) in plain language,
- **recommends the hardware and the software** to match,
- and helps you actually get it — including pulling options from marketplaces.

In short: describe what you want your homelab to do, and it figures out the box
and the stack for you. This repo is the ground floor of that — the building
blocks the rest will grow on.

## Status

**v0.1 — foundations.** Right now this is a growing, curated collection of
ready-to-run self-hosted services, one added at a time. Expect placeholders and
rough edges; structure and content will change as the project grows. The API,
the assistants and the guided setup are still ahead.


## Getting started

Everything here runs on **Docker** with the **Compose v2** plugin. You only
install this once per machine; after that, running any service is the same three
commands everywhere.

### Install Docker

> You need a 64-bit Linux host. The commands below install Docker Engine **and**
> the Compose v2 plugin (the `docker compose` command, with a space).

**Quickest way — any distribution** (Docker's official script auto-detects your
system):

```bash
curl -fsSL https://get.docker.com | sh
```

Prefer to use your distribution's official repository? Pick your system:

<details>
<summary><b>Debian / Ubuntu</b> (apt)</summary>

```bash
# 1. Remove any old/unofficial versions (safe to ignore "not installed")
sudo apt-get remove -y docker docker-engine docker.io containerd runc 2>/dev/null

# 2. Add Docker's official repository
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Install Docker Engine + Compose plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

> On **Debian** replace both `.../linux/ubuntu` URLs with `.../linux/debian`.

</details>

<details>
<summary><b>Fedora / RHEL / Rocky / Alma</b> (dnf)</summary>

```bash
# 1. Add Docker's official repository
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

# 2. Install Docker Engine + Compose plugin
sudo dnf install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# 3. Start Docker now and on every boot
sudo systemctl enable --now docker
```

> On **RHEL / Rocky / Alma** replace `.../linux/fedora/...` with `.../linux/rhel/...`.

</details>

<details>
<summary><b>Arch Linux</b> (pacman)</summary>

```bash
# Install Docker + the Compose plugin
sudo pacman -S --needed docker docker-compose

# Start Docker now and on every boot
sudo systemctl enable --now docker
```

</details>

### After installing (all systems)

```bash
# Start Docker on boot (no-op if already enabled)
sudo systemctl enable --now docker

# Run docker without sudo — then LOG OUT and back in for it to take effect
sudo usermod -aG docker "$USER"

# Verify everything works
docker --version
docker compose version
docker run --rm hello-world
```

> **Compose v1 vs v2.** Modern Docker uses `docker compose` (a space, v2 plugin).
> Some older setups only ship the legacy standalone `docker-compose` (a hyphen,
> v1). The commands are otherwise identical — just swap `docker compose` →
> `docker-compose` if that's what your system has.

### Run a service

Each service folder is self-contained — copy the examples, fill in your values,
and bring it up:

```bash
cd <category>/<service>
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env to taste, then:
docker compose up -d
```


## What's here now

- Ready-to-run service stacks (Docker Compose), each self-contained.

## Coming later

- Step-by-step guides written for non-technical users
- Curated bundles — e.g. "starter homelab", "media homelab"
- A guided, budget-aware setup — and eventually the AI-assisted builder above


## Curious about my actual setup?

Kujira Box is the polished, curated front. If you want to see the real thing —
my running homelab, the experiments with networking, VPS and my personal
server, the messy notes and the stacks I actually run — it all lives in my main
repo:

**→ [DevOps_lab](https://github.com/ThomasJager99/DevOps_lab)**

That's where the building blocks here come from.


## License

Licensed under the **Apache License 2.0** — see [LICENSE](LICENSE).

---

*Built in the open, growing as I go. ⭐ Star it to follow along.*

