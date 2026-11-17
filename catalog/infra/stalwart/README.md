# Stalwart Mail

Stalwart is a **modern, all-in-one mail server**: SMTP, IMAP, POP3 and JMAP, a
built-in spam/phishing filter, DKIM/SPF/DMARC/ARC, encryption, and a clean web
admin — all in a single Rust binary, in one container. It's the ambitious finale
of this series: your *own* email, on your *own* domain, on your *own* box. No
Google, no Microsoft reading your mail.

A straight-talking note up front: **this is the hardest thing to self-host
well.** The container is easy. Deliverability is not — getting the wider world
to accept and trust your mail depends on DNS and network conditions outside this
file. Treat it as a serious project, not a five-minute `up -d`.


<p align="center">
  <img src="../../../assets/logos/stalwart.svg" width="180">
</p>

<br>


## Official container image

[`stalwartlabs/stalwart`](https://github.com/stalwartlabs/stalwart) — the
official image. Pin a release (here `v1.1.0`).

## Deployment Notes

### First run

```bash
docker compose up -d
docker compose logs stalwart | grep -i "administrator\|password"
```

Stalwart generates an **admin password on first boot and prints it in the
logs** — grab it, open the web admin at `http://<host>:8080`, log in as the
admin account, and change it. Configure domains, accounts and settings from
there.

### The part that actually decides success: DNS + network

For mail you *send* to be accepted (and not land in spam), you need, on public
DNS for your domain:

- **A/AAAA** record for `mail.example.com` → your server IP.
- **MX** record for the domain → `mail.example.com`.
- **SPF** (TXT): authorize your server to send.
- **DKIM** (TXT): Stalwart generates the key — publish the matching record.
- **DMARC** (TXT): policy for the above.
- **PTR / reverse DNS**: your IP must resolve back to `mail.example.com`. Set
  this at whoever owns the IP (your VPS/ISP) — you can't fake it.

And the network conditions:

- **Port 25 outbound/inbound must be open.** Most residential ISPs **block 25** —
  a home connection usually can't run real mail. A VPS with a clean static IP is
  the normal home for this.
- **The IP must not be on a blocklist** (many residential/VPS ranges are). Check
  before committing.

If you can't meet these, you can still run Stalwart internally, or send outbound
via a **smarthost/relay** (a provider that handles the reputation for you).

### Exposure & TLS

- **Mail protocol ports (25/465/587/143/993)** are public by necessity — other
  servers and your mail clients connect to them directly. `LISTEN_ADDR` does
  **not** apply to these.
- **The web admin / JMAP port (8080)** *does* honor `LISTEN_ADDR` — set
  `127.0.0.1` and reach it through your reverse proxy with TLS (hardening entry
  09). Don't leave the admin panel open on the LAN.
- Stalwart can obtain its own Let's Encrypt certs for the mail ports, or you can
  provide certs — see its docs.

## Example Use

```bash
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# edit .env — set MAIL_HOSTNAME to your real FQDN, LISTEN_ADDR for the admin port
docker compose up -d
docker compose logs stalwart | grep -i password   # get the admin password
# open http://<host>:8080 → log in → add your domain → publish the DNS it shows you
```

## Thanks

Thanks to [Stalwart Labs](https://github.com/stalwartlabs/stalwart) for bringing
a genuinely modern, all-in-one mail server to the self-hosted world.

## Links

- Docs: https://stalw.art/docs/
- Source: https://github.com/stalwartlabs/stalwart
- Deliverability / DNS setup: https://stalw.art/docs/server/dns
