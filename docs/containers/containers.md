# Containers

## What Is It?

A container is **a normal Linux process** wearing namespaces (its own view: process list, network, filesystem, hostname) and cgroups (its resource rations) — packaged with a filesystem snapshot so it runs identically anywhere:

```text
container = process
           + namespaces   (isolation — what it sees)      ← Linux phase
           + cgroups      (limits — what it gets)         ← Linux phase
           + image layers (filesystem — what it stands on)
```

You built this by hand in the Linux phase lab. Docker adds packaging, distribution, and UX — not the isolation itself.

## Why Does It Exist?

The deployment problem containers solved wasn't isolation — VMs had that. It was **the works-on-my-machine problem between environments**: dev's laptop, CI, staging, prod — each with different JDKs, system libs, configs. Containers ship **the filesystem with the process**: the app *and its entire dependency closure* as one immutable unit.

```text
Before: app + hope (environment assumptions)
After:  app + environment (as an immutable artifact — the CI/CD pages' artifact, now executable)
```

That's why containers slotted perfectly into the pipeline story: image = artifact, registry = repository, deploy = run the artifact. CI/CD adopted containers because they made promotion *byte-identical* across environments.

## Layer 1 — Simple Explanation

A container is a **food truck**: kitchen (dependencies), chef (process), power and water hookups (kernel-shared resources), and a ration card (cgroups) — it parks anywhere with hookups (any Linux kernel) and serves the same menu. A VM, by contrast, is a **whole restaurant building** — own plumbing, heating, foundation (kernel): safer isolation, far heavier to build and move.

## Layer 2 — Engineer's View

**VM vs container — the comparison that explains everything:**

| | Container | VM |
|---|---|---|
| Virtualizes | OS view (shared kernel) | hardware (own kernel) |
| Size | MBs | GBs |
| Boot | ms–s | 30s+ |
| Density | 100s per host | ~10s |
| Isolation | kernel-boundary strength | hypervisor strength |
| Implication | multi-tenant strangers → VMs; your own workloads → containers | |

**What "shared kernel" means in practice — the security sentence:** container isolation is as strong as syscalls are safe. Escape = a kernel bug reached through a syscall. Hence: capabilities dropped, seccomp filters, user namespaces, non-root — the hardening stack (Security phase). "Containers are not VMs" is literally this row of the table.

**The image contract (next page deep-dives):** an image is an *ordered set of filesystem layers* (overlayfs!) plus metadata (entrypoint, env, ports). Layers are content-addressed and shared — 40 containers on a host download a base layer once. Immutability: you never patch a running container; you build a new image and redeploy — deployment *is* replacement (immutable infrastructure, IaC phase).

**Processes first — the operational mindset shift:**

```bash
docker run redis        # ← not "starting a VM": fork/exec of redis-server with 7 namespaces
ps aux | grep redis     # it's right there, a process on your host
kill -TERM <pid>        # works. (kubectl delete → TERM → grace → KILL: the Signals page)
```

Consequences: containers die like processes (fast, expected); supervision/restart is the *platform's* job (systemd did this for VMs; orchestrators do it for containers); and logs are just stdout/stderr (the journald model generalized).

**When containers are wrong:** kernel-adjacent work (custom modules), strict multi-tenancy of strangers, GUI/legacy apps with weird hardware assumptions — the escape hatch back to VMs exists for reasons.

## Real-World Example (DevOps flavored)

The pipeline story you already operate, now conceptually complete:

```yaml
# CI: source → image (the artifact)
docker build -t registry/shop/payments:2.1.0 .
docker push registry/shop/payments:2.1.0      # artifact repository (immutable tag!)
# CD: deploy = replace processes running that artifact
kubectl set image deploy/payments payments=registry/shop/payments:2.1.0
```

The debugging flex of understanding process-ness: `nsenter -t <pid> -n ss -tlnp` from the host when exec into the container is broken (Linux phase — you have this now).

## Common Mistakes

- Treating containers as light VMs (SSH-ing in, patching live, "restarting into" fixes)
- Storing state in the container filesystem — it vanishes with the container (volumes are the answer, K8s phase)
- Running as root with full capabilities because "it's containerized"
- Giant images (1.5 GB "ubuntu + everything") — slow pulls = slow deploys/scaling; multi-stage builds fix this
- Confusing image immutability with deployment safety — a bad image is still immutable

## Mental Model

> A container is a **food truck**: the process is the chef, namespaces are the truck's walls (what's inside is all it sees), cgroups the ration card, and the image the truck's pre-stocked interior. Any city with hookups (a Linux kernel) hosts the identical truck — and you never repaint a truck in service; you build a new one and drive it into the old one's spot.

## Remember This

1. Container = process + namespaces + cgroups + image layers — no magic, you built one
2. Solved environment-reproducibility (the artifact contract), not just isolation
3. Shared kernel: density and speed in exchange for kernel-boundary security — hardening stack compensates
4. Immutable images: patching = rebuilding; deployment = replacement
5. Containers are processes: they die, they're supervised, logs are stdout
6. Image size is deploy latency — multi-stage builds, slim bases

## One Sentence

A container is an ordinary Linux process isolated by namespaces and rationed by cgroups, packaged with an immutable filesystem snapshot so that the artifact running in production is byte-identical to the one CI tested.

## Knowledge Check

1. Why did CI/CD adopt containers so quickly? Answer in terms of the artifact contract.
2. Name the exact kernel features Docker orchestrates for `docker run`.
3. What does "patch a container" actually mean, mechanically?
4. Why are strangers' workloads on VMs, not containers?

## Further Reading

- Namespaces / cgroups pages (you've been there — this page *is* their payoff)
- Next: [Docker & Images](docker-images.md)

---

**← Previous:** [Multi-Region & Multi-Cloud](../cloud/multi-region-cloud.md)
**Next:** [Docker & Images](docker-images.md) →
**Related:** [Namespaces](../linux/namespaces.md) · [cgroups](../linux/cgroups.md)
