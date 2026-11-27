# Hardening 🛡️

A running collection of **container & homelab security** techniques — one topic
at a time, the same way the service catalog grows. Each entry is a short,
practical note: *what the risk is, how to defend against it, and how to verify
it actually worked.*

The guiding idea: **a container is a boundary, not a guarantee.** Most of these
stacks run on hardware you own and some of them face the internet. This rubric
is how we make sure that even a compromised or malicious image can do as little
as possible — and how to check what you run *before* you run it.


<p align="center">
  <img src="../assets/logos/security_logo.png" width="180">
</p>

<br>


## How it's organised

Three blocks. They're interleaved day to day (so it's not ten days of one
flavour), but they group like this:

- **A — Protect containers from the host.** Even internet-facing ones.
  Least-privilege compose, the Docker socket, network isolation, reverse
  proxy + auth, rootless engines, host firewall.
- **B — Vet what you download.** Supply-chain: image provenance, CVE scanning,
  pinning by digest, signatures, inspecting layers.
- **C — Sandboxes for untrusted code.** VMs vs containers, disposable test
  environments, running something sketchy without getting burned.

## Format of each entry

Every topic is its own folder with a `README.md` (the explanation) and, where
code makes it concrete, an example file you can copy:

```
hardening/
├── README.md                     ← this index
└── <nn>-<topic>/
    ├── README.md                 ← the risk → the fix → how to verify
    └── <example>.yml | *.sh | *.tf   ← a working example (when useful)
```

Compose/YAML and shell carry most of it. A few topics (throwaway test VMs, host
provisioning) ship a **Terraform** example instead — that's where "declare the
infrastructure" is the right tool, as opposed to hardening a container, which
lives in the compose file.

## Entries

1. [Threat model & baseline hardening](01-threat-model/) — is a container a security boundary? + the first round of defense, as one copy-paste block.

2. [Drop root](02-drop-root/) — run containers as a non-root user; the single highest-value line in the baseline block.

3. [Capabilities](03-capabilities/) — `cap_drop: [ALL]` + selective add-back + `no-new-privileges`: strip root's leftover superpowers.

4. [Read-only rootfs + tmpfs](04-read-only-rootfs/) — freeze the container's filesystem; hand back only the scratch paths it needs.

5. [Resource & PID limits](05-resource-limits/) — stop one container starving the host: `mem_limit` / `cpus` / `pids_limit`.

6. [Network segmentation](06-network-segmentation/) — `internal: true` backends with no egress: cut off exfiltration and phone-home.

7. [Host firewall & the Docker-bypasses-UFW trap](07-docker-ufw-trap/) — why `ufw status` lies about published container ports, and the three fixes.

8. [The Docker socket is root](08-docker-socket/) — why mounting docker.sock = handing over the host, and the socket-proxy that fixes it.

9. [Reverse proxy + TLS in front of everything](09-reverse-proxy-tls/) — one hardened front door instead of a dozen exposed app ports.

10. [Forward-auth / SSO in front of everything](10-forward-auth-sso/) — one login layer that gates requests before they reach the app.

11. [Blocking brute-force: fail2ban & CrowdSec](11-fail2ban-crowdsec/) — watch logs, ban the IPs that keep knocking.

12. [Rootless Docker](12-rootless-docker/) — run the daemon as a normal user so an escape lands unprivileged, not on host root.

13. [Podman: rootless & daemonless by default](13-podman/) — the engine that starts where hardened Docker ends up.

14. [Secrets: beyond the plaintext .env](14-secrets/) — where a plaintext .env leaks, and the ladder up (Docker secrets → SOPS+age → secrets manager).

15. [Image provenance: do you trust this image?](15-image-provenance/) — who made it, from what, and how — the check before you pull.

16. [Docker Scout: the scanner already on your machine](16-docker-scout/) — a second-opinion CVE scanner built into the Docker CLI, with concrete base-image upgrade advice.

17. [A second opinion with Grype](17-grype/) — run an independent CVE scanner (different database) and see where it disagrees with Scout; the one built for CI gating.

18. [Know what's inside: SBOMs with Syft](18-syft/) — generate a machine-readable ingredient list of an image; the foundation for fast scanning and incident response.

19. [Running untrusted code in micro-VMs: Docker Sandboxes](19-docker-sandboxes/) — give risky code its own kernel; the real answer to the docker-socket problem (#08).



## Using these

The hardening blocks slot straight into the service stacks in
[`docker_compose_services/`](../docker_compose_services/). Where a service's
compose shows a commented hardening section, these entries are the *why* and
the *how far you can push it* behind that section.

> Homelab-honest: none of this is "enterprise zero-trust." It's the realistic
> set of moves that cut your blast radius by a lot, for a single person running
> their own box.
