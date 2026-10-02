# OCI & Container Runtimes

## What Is It?

The **Open Container Initiative** (2015, post-"Docker vs the world" schism) standardized the two interfaces that matter:

1. **Image spec** — what an image *is* (layers, manifest, config): any build tool → any registry → any runtime
2. **Runtime spec** — how a container *starts* (bundle = rootfs + config.json): `runc` is the reference implementation

The resulting stack running on every modern node:

```text
docker / podman / nerdctl          UX layer
      ↓
containerd / CRI-O                daemon: image pulls, lifecycle, snapshot mgmt
      ↓
runc                               actually creates namespaces/cgroups and execs
                                    (your Linux-phase lab, industrialized)
```

K8s dropped Docker support (2021's "dockershim removal") with zero user impact *because of this standardization*: kubelet speaks **CRI** to containerd/CRI-O, and images were already OCI images.

## Why Does It Exist?

Vendor lock-in fear. By 2015, one company controlled the image format, the runtime, and the registry protocol. The industry (including Docker's partners) extracted the two specs into a neutral foundation — and the result was an explosion of *interoperable* tooling: Podman (daemonless, rootless-friendly), Kaniko/BuildKit (building without Docker-in-Docker), Buildah, firecracker microVMs (AWS Lambda/Fargate), Kata (VM-isolated containers).

The lesson worth keeping: **standardize the interface, compete on implementation** — the same move as HTTP (browsers), SQL (databases), and JVM (languages).

## Layer 1 — Simple Explanation

OCI split the container world into **LEGO-standardized bricks**: images built by any tool fit registries run by anyone and run on runtimes made by anyone. Docker, Podman, and Kubernetes are different hands playing with the same bricks — the *shape of the bricks* is the standard.

## Layer 2 — Engineer's View

**What each layer actually contributes (and where bugs live):**

| Layer | Responsibilities | Ops relevance |
|---|---|---|
| runc | namespace/cgroup setup, exec, lifecycle | CVEs here = escapes |
| containerd | image management, snapshots (overlayfs), CRI API | node-level container debugging |
| kubelet | pod-level lifecycle via CRI | pod events, sandbox issues |

**The security spectrum of runtime implementations** (same OCI spec, different isolation depths):

| Runtime | Isolation | Trade |
|---|---|---|
| runc (default) | kernel namespaces | fastest, kernel-boundary trust |
| Kata Containers | microVM per container | VM-strong isolation, ~100s ms boot |
| gVisor (runsc) | userspace kernel shim | strong syscall filtering, some syscall overhead |

Multi-tenant/hot workloads pick Kata/gVisor via RuntimeClass — the "containers aren't VMs" caveat (Containers page) answered with pluggable runtimes rather than abandoning containers.

**Rootless containers**: runc/Podman with user namespaces (Linux page) — containers run without root on the host, at some capability cost. The future default for developer machines and hardening path for CI agents.

**Building without Docker-in-Docker:** CI agents that need to build images face the "Docker socket mounted = root on host" problem (Filesystems page). Standard-compliant builders fixed this:

- **BuildKit** — modern Docker/Podman build engine (parallel layers, cache mounts, multi-platform)
- **Kaniko / Buildah** — build in an unprivileged container, no daemon, no socket

**Image manifest v2/OCI artifacts:** registries now carry more than images — Helm charts, SBOMs, signatures (cosign) all ride the OCI artifact protocol. One distribution mechanism for the whole supply chain (Security phase builds on this).

## Real-World Example (DevOps flavored)

Your Kubernetes node's process tree, decoded:

```bash
crictl ps                          # CRI view: containers on the node
pstree -p $(pidof containerd) | head -5   # runtime-shim → runc-spawned processes
cat /var/lib/containerd/io.containerd.../config.json | head  # the runtime spec bundle:
                                                    # namespaces list, cgroupsPath, mounts
```

And the CI pattern that closed the socket hole:

```yaml
# Before: mount /var/run/docker.sock into the agent = root escape    ❌
# After: Kaniko step, unprivileged, pushes to registry               ✅
- image: gcr.io/kaniko-project/executor
  args: ["--dockerfile=/workspace/Dockerfile", "--destination=reg/shop/app:${SHA}"]
```

## Common Mistakes

- Confusing "K8s dropped Docker" with "K8s can't run Docker images" (they're OCI images — always could)
- Docker socket mounted in CI/pods for convenience — the perennial escape hatch left open
- Reaching for microVM runtimes everywhere (paying boot cost needlessly) or nowhere (ignoring multi-tenant risk)
- Building with deprecated `docker build` paths instead of BuildKit (missing cache mounts/parallelism)

## Mental Model

> OCI is the **LEGO standard for the container world**: image = brick shape, runtime = clicking mechanism. Docker was the original toy company; containerd/runc are the bricks inside every set; Podman, Kata, and Kaniko are specialist manufacturers — all interchangeable because the standard, not any vendor, defines the interface.

## Remember This

1. OCI standardized image + runtime specs; CRI standardized the kubelet-runtime interface
2. Stack: UX → containerd/CRI-O → runc (which does what your hand-lab did)
3. RuntimeClass options (Kata, gVisor) buy VM-grade isolation per workload
4. BuildKit/Kaniko/Buildah build images without privileged daemons — no docker.sock
5. OCI artifacts carry charts, SBOMs, signatures — one supply-chain distribution protocol
6. Standardize interfaces, compete on implementations — the industry pattern to reuse

## One Sentence

The OCI standards made images and runtimes interchangeable interfaces, so containers became infrastructure you can assemble from any vendor's parts — with isolation depth (runc, gVisor, Kata) chosen per workload rather than per vendor.

## Knowledge Check

1. Why did removing dockershim not break anyone's images?
2. Your CI needs mounts of the Docker socket. What's the risk and the standard-compliant fix?
3. When would you schedule a pod with a Kata RuntimeClass, and what do you pay?
4. What rides on the OCI artifact protocol besides container images?

## Further Reading

- [opencontainers.org](https://opencontainers.org) — the specs (short reads)
- containerd docs — architecture diagrams

---

**← Previous:** [Docker & Images](docker-images.md)
**Next:** [Registries](registries.md) →
**Related:** [Artifact Repositories](../cicd/artifact-repositories.md) · [SBOM & Supply Chain](../security/sbom-supply-chain.md)
