# 19 — Running untrusted code in micro-VMs: Docker Sandboxes & friends

Back in entry 08 we hit a wall: some tools *must* have the Docker socket or root,
and mounting the host's `docker.sock` into a container hands that container root
on the host. Entries 09–18 reduced risk around the edges, but none of them change
the fundamental problem — **a normal container shares the host's kernel.** Namespaces
and cgroups are a *thin* wall; a kernel bug or a mounted socket goes straight
through it.

This entry is the real answer for "I genuinely need to run something I don't
fully trust": don't isolate it *better* at the namespace level — **give it its
own kernel.**

## Container vs. micro-VM: thin wall vs. thick wall

| | Normal container | Micro-VM sandbox |
|---|---|---|
| Kernel | **shared** with the host | **its own**, separate kernel |
| Isolation | namespaces + cgroups (OS-level) | hardware virtualization (KVM) |
| Escape = | potentially host root | trapped inside a throwaway VM |
| Boot time | milliseconds | ~tens–hundreds of ms |
| Overhead | almost none | some RAM + CPU per VM |

A container is a **process** the host kernel pretends is alone. A micro-VM is a
tiny **real machine** with its own kernel, booted in a fraction of a second by a
minimal hypervisor. Escape the container and you might own the host; escape into
the micro-VM's kernel and you're still stuck in a disposable box.

## Docker Sandboxes (`sbx`) — this is a real Docker product now

As of 2026 Docker ships **Docker Sandboxes**: each sandbox is a micro-VM with its
**own Linux kernel, its own filesystem and network, and a private Docker Engine
inside it**. On Linux it's a standalone CLI, `sbx`:

```bash
# requirements: Ubuntu 24.04+, KVM available, your user in the `kvm` group
sudo apt install docker-sbx

sbx run --name agent claude     # run a coding agent inside a micro-VM
sbx run shell                   # a bare micro-VM shell to poke around
```

Here's the payoff for entry 08's socket problem: the Docker socket the thing
inside sees belongs to the **private daemon in the VM** — the host's daemon and
socket **stay outside the micro-VM the whole time.** So you can safely "grant the
Docker socket" to an agent: even if it goes fully rogue and roots its own Docker,
the blast radius is one disposable VM, not your machine.

Docker built this specifically to run **AI coding agents** (Claude Code and
friends) unsupervised-but-safely — full root/Docker inside, host untouched, with
a default-deny network and no access to your `~/.ssh` or host socket.

## The wider landscape (when `sbx` doesn't fit)

Same idea, different tools — useful on servers, CI, or where you want a drop-in
runtime:

- **Kata Containers** — an OCI runtime that transparently runs each container in
  a lightweight VM. Drop-in: `docker run --runtime=kata ...`. Backed by QEMU or
  Firecracker. The classic "containers, but in micro-VMs."
- **Firecracker** — AWS's minimal virtual-machine monitor; micro-VMs boot in
  ~125 ms. It's what powers AWS Lambda and Fly.io. Kata can use it as a backend.
- **gVisor (`runsc`)** — *not* a VM: a user-space kernel that intercepts a
  container's syscalls and handles them itself, so the real host kernel is barely
  touched. Lighter than a VM, a different trade-off. (You met it today: **Grist**
  uses gVisor to sandbox the Python in your formulas.)
- **Sysbox** — a runtime that lets you run Docker/systemd *inside* a container
  safely, using user-namespaces rather than a VM. Great for dev-in-a-container.

Rough rule: **gVisor** for a lighter syscall-level sandbox, **Kata/Firecracker
/`sbx`** when you want a true separate kernel, **Sysbox** for nested Docker.

## The catch: hardware virtualization

Micro-VMs need **KVM** (hardware virtualization). On bare-metal Linux or a VM
with nested virtualization enabled, fine. On some cheap VPSes nested virt is off,
and `sbx`/Kata/Firecracker simply can't start — there, **gVisor** is the fallback
that still works, since it's not a VM. There's also a small RAM/CPU cost per
micro-VM, so you use this for the *untrusted* workloads, not every container.

## How to verify (try it now)

```bash
# if you're on Ubuntu 24.04+ with KVM:
sbx run shell
#   inside, prove it's a different kernel from the host:
uname -a           # different kernel version/build than `uname -a` on the host
#   start docker inside — it's a private daemon, not the host's:
docker info        # a separate engine, isolated in this micro-VM
exit               # the whole VM is thrown away
```

If `sbx` won't start, check `kvm-ok` (package `cpu-checker`) — no KVM means no
micro-VM, and gVisor is your path instead.

## Today's example

Today's service, **Grist**, is a live, unglamorous example of this principle at
work: every time you type a Python formula, Grist runs it inside a **gVisor**
sandbox (`GRIST_SANDBOX_FLAVOR=gvisor`) so a malicious formula can't escape into
the container, let alone the host. You don't have to build the isolation
yourself — a well-designed app already sandboxes the untrusted part (your
formula) the same way `sbx` sandboxes an untrusted agent. Same idea, two scales.

## Links

- Docker Sandboxes: https://www.docker.com/products/docker-sandboxes/ · https://www.docker.com/blog/docker-sandboxes-run-claude-code-and-other-coding-agents-unsupervised-but-safely/
- `sbx` on Linux (Arm install guide): https://learn.arm.com/install-guides/sbx/
- Kata Containers: https://katacontainers.io/
- Firecracker: https://firecracker-microvm.github.io/
- gVisor: https://gvisor.dev/
