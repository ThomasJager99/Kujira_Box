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

<!-- add one line per topic as it lands -->

1. [Threat model & baseline hardening](01-threat-model/) — is a container a
   security boundary? + the first round of defense, as one copy-paste block.

## Using these

The hardening blocks slot straight into the service stacks in
[`docker_compose_services/`](../docker_compose_services/). Where a service's
compose shows a commented hardening section, these entries are the *why* and
the *how far you can push it* behind that section.

> Homelab-honest: none of this is "enterprise zero-trust." It's the realistic
> set of moves that cut your blast radius by a lot, for a single person running
> their own box.
