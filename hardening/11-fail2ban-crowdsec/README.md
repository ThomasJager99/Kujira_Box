# 11 — Blocking brute-force: fail2ban & CrowdSec

Entries 09–10 gave exposed services a locked front door. This entry deals with
the attackers who keep **knocking** — the endless automated login attempts,
credential-stuffing and scanning that any internet-facing service attracts. The
fix is to *watch the logs and ban the IPs that misbehave*.

## The risk: exposure means constant automated attack

The moment anything of yours is reachable from the internet — SSH, a reverse
proxy, a mail server (Stalwart, yesterday), a login page — bots find it within
minutes and start trying. Thousands of attempts a day is normal. Each one is:

- a **credential-stuffing** guess against your logins,
- a **vulnerability scan** probing for known-exploitable paths,
- noise that can tip into a **resource drain** (a login endpoint hit hard).

Strong passwords and MFA (entry 10) stop them from *succeeding*, but they keep
*trying* forever, and every so often a new CVE turns "trying" into "getting in."
You want to **cut them off** after a few failures — stop reading the attempts
and start dropping the source.

## Two tools, two philosophies

### fail2ban — the classic, local, simple

fail2ban tails your log files, matches failure patterns (a "jail" per service),
and when an IP crosses a threshold it adds a firewall rule to **ban** it for a
while.

- **Local only** — it reacts to *your* logs, *your* bans. Nothing shared.
- **Battle-tested, tiny, everywhere.** The default for protecting host **SSH**.
- You write/enable jails (sshd, nginx, your app's auth log) and set
  bantime/findtime/maxretry.

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled  = true
maxretry = 4           # 4 failures...
findtime = 10m         # ...within 10 minutes...
bantime  = 1h          # ...= banned for an hour (use -1 for permanent)

[nginx-http-auth]
enabled  = true
```

Best fit: **host-level SSH**, and simple per-service jails on a single box.

### CrowdSec — modern, behavioral, collaborative

CrowdSec also parses logs and bans, but adds two things:

- **A shared reputation network.** When CrowdSec instances worldwide see an IP
  attacking, that IP's reputation is shared — so you can block known-bad IPs
  *before* they ever hit you, not just after they fail on your box.
- **Scenarios + bouncers split.** The engine *detects* (scenarios: brute-force,
  scanning, scraping); separate **bouncers** *enforce* (at the firewall, at
  nginx/Traefik, at the reverse proxy). You can detect in one place and block at
  the edge.

```yaml
# a CrowdSec engine + a firewall bouncer, in Docker
services:
  crowdsec:
    image: crowdsecurity/crowdsec:latest
    environment:
      COLLECTIONS: "crowdsecurity/nginx crowdsecurity/sshd"
    volumes:
      - /var/log:/var/log:ro          # read the logs to analyze
      - crowdsec-data:/var/lib/crowdsec/data
    # a bouncer (e.g. crowdsec-firewall-bouncer or the Traefik/nginx bouncer)
    # then enforces the decisions at your edge.
```

Best fit: a reverse-proxy-fronted homelab with several exposed web apps — detect
across all of them, enforce at the proxy, and benefit from the shared blocklist.

## Which to use

- **Just protecting SSH on one host?** fail2ban. Five minutes, done.
- **Several web apps behind a reverse proxy, want crowd-sourced blocking and
  edge enforcement?** CrowdSec.
- Running both is fine (fail2ban on SSH, CrowdSec on the web edge) — they don't
  conflict.

Either way, the principle is the same and it's the point of this entry: **turn
repeated failure into an automatic ban.**

## Don't lock yourself out

- **Allowlist your own IPs/subnets** (`ignoreip` in fail2ban;
  `cscli decisions`/allowlists in CrowdSec) so a fat-fingered password doesn't
  ban *you*.
- Keep bantimes sane (an hour, escalating) rather than instant-permanent while
  you're still testing.
- If you administer over a VPN (your MikroTik/WireGuard setup), allowlist that
  range and you're safe even if a jail misfires.

## How to verify

```bash
# fail2ban: is the jail active and has it banned anyone?
sudo fail2ban-client status sshd
# -> shows currently banned IPs + totals

# generate a few bad SSH logins from a THROWAWAY IP, then re-check the status
# -> that IP should appear in "Banned IP list"

# CrowdSec: what has it decided, and are bouncers connected?
sudo cscli decisions list        # active bans
sudo cscli bouncers list         # enforcers that are wired up
sudo cscli metrics               # what it's parsing and acting on
```

## Today's example

NocoDB (today's service) has a login — exactly the kind of web endpoint that
draws brute-force once exposed. The layered answer from the last few entries:
reverse proxy + TLS (09) → forward-auth/MFA so guesses can't succeed (10) →
**fail2ban/CrowdSec to ban the IPs that keep guessing (11)**. Depth, not a
single wall. And remember yesterday's Stalwart: a public mail server is a
*magnet* for brute-force on SMTP/IMAP — CrowdSec has collections specifically
for mail.

## Links

- fail2ban: https://github.com/fail2ban/fail2ban
- CrowdSec: https://www.crowdsec.net/ · https://docs.crowdsec.net/
- CrowdSec hub (collections/bouncers): https://app.crowdsec.net/hub
