# 07 — The host firewall & the Docker-bypasses-UFW trap

This is the entry that catches almost everyone out. You set up UFW, you "deny
everything by default," you feel safe — and your Docker containers are wide open
to your whole network anyway, because **Docker quietly punches through UFW**.
If you run exposed services on a box with a firewall, read this one twice.

## The risk: your firewall isn't filtering what you think

UFW (the friendly front-end to `iptables`/`nftables`) adds its rules in one
place. **Docker adds its own `iptables` rules in a different place** — the
`DOCKER` chain — and manipulates the `nat`/`FORWARD` tables directly when you
publish a port. Docker's rules are evaluated **before** UFW's `INPUT` rules ever
get a say for forwarded/published container traffic.

The practical result:

```yaml
ports:
  - "5000:5000"      # you meant "expose on the host"
```

That `ports:` mapping binds to `0.0.0.0` by default and Docker opens it **to
every machine on your LAN** — even if UFW says `deny incoming`. Your `ufw status`
looks locked down; `nmap` from another device finds the port wide open. People
have put databases and admin panels "behind UFW" this way and had them reachable
across the whole network (or the internet, if the host is public).

So: **UFW does not protect published Docker ports.** You have to handle it
deliberately.

## Fix 1 (simplest): only publish to localhost

If a service only needs to be reached by the reverse proxy or by you via an SSH
tunnel — bind it to `127.0.0.1`, not `0.0.0.0`:

```yaml
ports:
  - "127.0.0.1:5000:5000"   # host-only; NOT reachable from the LAN
```

Now the port exists only on the loopback interface. Your reverse proxy (on the
same host, or on a shared docker network) reaches it; nobody else can. This is
the move you want for *most* homelab services — expose only the proxy's 80/443
to the network, keep everything else on `127.0.0.1` or on an internal docker
network with **no** `ports:` at all (entry 06).

## Fix 2 (cleanest): don't publish, use a shared network

If only another container (your reverse proxy) needs the service, don't publish
a port **at all** — put both on a shared docker network and let the proxy reach
it by name. Zero host exposure, nothing for a firewall to filter. This is what
today's stack does with the browser sidecar.

## Fix 3 (when you DO need LAN exposure): make UFW and Docker cooperate

For ports you genuinely need open to the LAN, use **`ufw-docker`**, which inserts
rules into the `DOCKER-USER` chain (the one spot Docker leaves for you, evaluated
before its own rules):

```bash
# install the helper
sudo wget -O /usr/local/bin/ufw-docker \
  https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
sudo chmod +x /usr/local/bin/ufw-docker

sudo ufw-docker install      # patches /etc/ufw/after.rules to respect Docker
sudo systemctl restart ufw

# now allow a specific container's port explicitly:
sudo ufw-docker allow changedetection 5000
```

After `ufw-docker install`, published container ports are **denied by default**
and you allow them per-container — UFW is finally telling the truth.

## A baseline host firewall (the normal part)

Beyond the Docker gotcha, the ordinary UFW baseline still applies to the host
itself (SSH, the proxy):

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp          # SSH (ideally limited to your IP / via VPN)
sudo ufw allow 80,443/tcp      # the reverse proxy — the ONLY public web ports
sudo ufw enable
sudo ufw status verbose
```

The goal: the **only** things reachable from the network are SSH and your
reverse proxy. Every app sits behind the proxy, on localhost or an internal
network — never exposed directly.

## How to verify (do this — don't assume)

```bash
# from ANOTHER machine on the LAN, scan the docker host:
nmap -p- <docker-host-ip>
# you should see ONLY the ports you intend (22, 80, 443) — NOT app ports

# check what's actually bound, and to which interface:
sudo ss -tlnp | grep -E 'docker|:5000'
# 127.0.0.1:5000  = good (host-only)
# 0.0.0.0:5000    = exposed to the LAN — fix it
```

`nmap` from a *different* device is the real test. `ufw status` lying to you is
the whole point of this entry — trust the scan, not the ruleset.

## Today's example

Changedetection.io publishes `5000` for its UI but keeps the Chrome sidecar
**unpublished** (Fix 2 — shared network, no `ports:`). In your own deploy, the
natural next step is Fix 1: change it to `127.0.0.1:5000:5000` and reach the UI
through your reverse proxy, so the only thing on the LAN is 80/443. The sidecar
needing `SYS_ADMIN` (entry 03) is exactly why you never want it reachable.

## Links

- The Docker + UFW problem (Docker docs): https://docs.docker.com/engine/network/packet-filtering-firewalls/
- `ufw-docker`: https://github.com/chaifeng/ufw-docker
- Bind published ports to an interface: https://docs.docker.com/engine/network/#published-ports
