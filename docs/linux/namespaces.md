# Namespaces

## What Is It?

Linux **namespaces** are a kernel feature that gives a process (and its children) **its own private view of a system resource** — as if it had its own copy of it.

| Namespace | Isolates | Since |
|---|---|---|
| **PID** | Process IDs (own tree, own "PID 1") | 2.6.24 |
| **NET** | Interfaces, routes, iptables, sockets | 2.6.29 |
| **MNT** | Mount points (own filesystem view) | 2.4.19 |
| **UTS** | Hostname | 2.6.19 |
| **IPC** | Shared memory, semaphores | 2.6.19 |
| **USER** | UID/GID mapping (inner root ≠ outer root) | 3.8 |
| **CGROUP** | cgroup view | 4.6 |

**A container is just a process launched with new namespaces (+ cgroups + overlayfs).** That's the entire secret. Docker adds convenience, not magic.

## Why Does It Exist?

The chroot era (1982+) faked only the filesystem view. The real problem: safely running untrusted/multi-tenant workloads on one kernel — you must limit what a process can *see*, not just what it can touch. Namespaces answered per-resource: fake network stack, fake process list, fake UIDs — each composable.

## Layer 1 — Simple Explanation

Namespaces are ** VR headsets for processes**. Put a headset (new namespace) on a process and it sees its own virtual world: it's "PID 1," it has its own hostname, its own network card, its own `/`. It shares the physical machine (kernel, CPU, RAM) with everyone else — but its *view* is private, and it can't even name things outside its world.

## Layer 2 — Engineer's View

**Working with them directly (the lab before the lab):**

```bash
unshare --net --pid --mount --uts --fork bash   # step into a minimal world
hostname container-1                            # UTS: private hostname
mount -t proc proc /proc                        # PID: fresh process table
ip link                                         # NET: just 'lo', and it's DOWN
# from outside (another shell):
lsns -t net,pid                                 # list namespaces on the host
nsenter -t <pid> -n -p                          # enter another process's namespaces
```

**The networking detail that explains container networking:** a fresh network namespace contains only a loopback — *down*. That's why Docker/K8s must, for every container: create a **veth pair**, drop one end into the new netns (as `eth0`), bridge the host end, and wire NAT rules (Linux Networking page). Container networking is namespace plumbing.

**The USER namespace — security's sharpest double edge:**

- Inside: UID 0 ("root"); outside: mapped to an unprivileged host UID
- This is how "rootless containers" work: inner root with no outer privileges
- Also historically the kernel's most-exploited attack surface (many CVEs) — some hardened hosts disable it

**Namespaces vs cgroups — the two-axis model to memorize:**

```text
namespaces = what a process SEES     (view/isolation)  → "my own world"
cgroups    = what a process GETS     (resources)       → "my own ration"
```

Containers = namespaces (isolation) + cgroups (limits) + overlayfs (image layers). Kubernetes then composes *shared* namespaces: a Pod shares net/IPC/UTS namespaces across its containers (that's why `localhost` works between them) — but not PID by default.

**What namespaces do NOT isolate:** the kernel itself, `/proc`-exposed kernel settings, hardware, and (crucially) syscalls — hence seccomp, capabilities, and the whole container-hardening conversation (Security phase). "Containers are not VMs" is precisely this sentence.

## Real-World Example (DevOps flavored)

The host-side view of a running Kubernetes node:

```bash
lsns -t net                          # one netns per pod (+ host + CNI ones)
nsenter -t <container-pid> -n ss -tlnp   # see the container's listeners from the host
crictl inspect <cid> | grep -i pid       # find the host PID of a container
# "kubectl exec into a broken container" fallback:
nsenter -t <pid> -m -n -p -- bash
```

This is also your forensic path when the container runtime is wedged but processes are alive.

## Common Mistakes

- Believing containers are lightweight VMs — they're namespaces on a *shared kernel* (isolation quality is entirely kernel-boundary quality)
- Root-in-container = root-on-host (without USER ns / dropped caps)
- Expecting PID 1 semantics inside a container to be automatic (Signals page: zombies, SIGTERM)
- Trying to "see" container processes from inside only — the host sees all; use that

## Mental Model

> Namespaces are **VR headsets bolted on per resource**: one for the process list, one for the network, one for the filesystem. A container is a process wearing seven headsets, living on a ration card (cgroups), with luggage (overlayfs). Remove one headset and the illusion breaks *for that resource only*.

## Remember This

1. Seven namespaces; each isolates *view* of one resource class
2. Container = process + namespaces + cgroups + overlayfs — no magic
3. NET ns starts with only `lo` (down) — veth/bridge/NAT is the plumbing you operate
4. Pod = containers sharing net/IPC/UTS namespaces (why localhost works inside a Pod)
5. USER ns = rootless containers, and the most security-sensitive one
6. The shared kernel is the boundary containers don't have — seccomp/caps do that job

## One Sentence

Namespaces give each process its own private view of system resources — processes, network, mounts, users — and combining them with cgroups and overlayfs is literally what a container is.

## Knowledge Check

1. Which three namespaces does a Pod share among its containers, and what does that make possible?
2. Why does a fresh network namespace have no internet, and what four-step plumbing fixes it?
3. How do rootless containers use USER namespaces?
4. What do namespaces NOT protect, and what kernel features fill that gap?

## Further Reading

- `man 7 namespaces`
- [LWN.net namespace series](https://lwn.net/Articles/531114/) — history & design
- The next page: [cgroups](cgroups.md), then [the hands-on lab](../projects/lab-container-by-hand.md)

---

**← Previous:** [Logs & Troubleshooting](logs-troubleshooting.md)
**Next:** [cgroups](cgroups.md) →
**Related:** [Container fundamentals](../containers/containers.md)
