# Container Networking & Storage

## What Is It?

How containers get **network identity** and **persistent state** — the two things a default container conspicuously lacks:

- **Networking**: veth pairs + bridges + NAT per container (Linux Networking page), orchestrated by Docker/CNI
- **Storage**: volumes — filesystem locations that *outlive* the container's writable layer

## Why Does It Exist?

Because the container contract is "isolated, ephemeral, immutable" — and real applications need the opposite: to be *reachable* and to *remember*. These two subsystems are the escape valves that let stateless-and-stateful reality coexist with the container model.

## Layer 1 — Simple Explanation

- **Networking**: each container gets a **private phone line** (veth) into the building switch (bridge) behind a shared outgoing line (NAT). Containers on the same bridge can call each other; the outside world sees only the building's number.
- **Storage**: the container is a **whiteboard** (wiped when it leaves); a volume is a **filing cabinet** bolted to the floor — any container working in that office reads/writes the same cabinet.

## Layer 2 — Engineer's View

**Networking — Docker's defaults, decoded (you know every piece already):**

```text
container netns: eth0 (veth end) 10.88.0.2
        │
host:    docker0 bridge 10.88.0.1 ── iptables MASQUERADE → eth0 (internet)
         └─ published port: -p 8080:80 = DNAT rule + proxy
```

The modes you choose among:

| Mode | What it does | Use |
|---|---|---|
| bridge (default) | private net + NAT | dev, simple hosts |
| host | no netns — container shares host stack | performance, Daemons |
| none | lo only | sandboxes, custom plumbing |
| overlay | VXLAN between hosts | multi-host (swarm/legacy) |

DNS: Docker's embedded server name-resolves containers on user-defined networks (`payments` "just works") — the preview of service discovery that K8s industrializes.

**Storage — the mount taxonomy:**

| Type | Backed by | Lifetime | The trap |
|---|---|---|---|
| writable layer | overlayfs copy-up | container only | anything saved here is gone |
| bind mount | host dir | host | *node-specific path* — breaks portability; SELinux issues |
| named volume | daemon-managed area | explicit | the default right answer on hosts |
| tmpfs | RAM | container | secrets-passing (or /dev/shm sizing) |

The K8s evolution (next pages): volumes → PV/PVC → StorageClasses — the same concept, decoupled from nodes by a provisioning layer.

**The statelessness discipline (architecture preview):** *containers must be disposable* — a container you're afraid to kill (because of its writable layer) is a pet, not cattle. All durable state goes to volumes/external services; the payoff is trivial horizontal scaling and fearless deploys. When an app can't comply (legacy), you reach for StatefulSets + PVs and accept the operational weight.

**Image vs runtime config:** networking and storage are *runtime decisions*, not image decisions — the same image runs with `--network=none` in CI and a production overlay later. Keep environment specifics out of images (env vars, mounts at run) — the twelve-factor idea that makes promotion byte-identical.

## Real-World Example (DevOps flavored)

The two classic production Docker mistakes, fixed:

```bash
# 1. "Our data disappeared after redeploy" — app wrote to /var/lib/data (writable layer)
docker run -v payments-data:/var/lib/data shop/payments:2.1.0     # ✅ named volume

# 2. "Container can't be reached / port conflicts everywhere"
docker run -p 127.0.0.1:8080:80 ...        # published but bound to loopback = local only
docker network create shop && docker run --network shop --name db ...
# on 'shop', app reaches it as host 'db' — embedded DNS
```

And the debugging flex: `nsenter -t <pid> -n ip addr` — see the container's veth from the host when the runtime is uncooperative.

## Common Mistakes

- Durable data in the writable layer — the redeploy amnesia
- Bind mounts with absolute host paths baked into runbooks — node-locked pets
- `-p 8080:80` everywhere in prod — unorchestrated port roulette; use real LBs/orchestrators
- Forgetting overlay/VXLAN MTU shrinkage (the Networking page's big-packet trap — returns at K8s scale)
- /dev/shm defaults breaking apps that need more shared memory (Oracle, Chrome)

## Mental Model

> Networking gives each container a **private line into the building switch behind one public number**; storage separates the **whiteboard you wipe** (container) from the **filing cabinet bolted to the floor** (volume). Apps that only need the whiteboard scale infinitely and die happily; apps that need the cabinet require the machinery of the next pages.

## Remember This

1. Container net = veth + bridge + NAT (the Linux page's plumbing, automated); host/bridge/none/overlay modes
2. Writable layer is scratch — volumes (named > bind) hold durable state
3. Disposable-container discipline: durable state external or in volumes, always
4. Network/storage are runtime decisions — images stay environment-agnostic
5. Embedded DNS on user networks previews K8s service discovery
6. VXLAN/overlay MTU remains the big-packet trap at scale

## One Sentence

Container networking wires isolated netns processes into bridges, NAT, and DNS so they become reachable, and volumes bolt durable storage onto otherwise disposable processes so state can outlive them.

## Knowledge Check

1. Trace a packet from container to internet — name each kernel mechanism (you've met all of them).
2. Why is a named volume safer than a bind mount for portability?
3. What makes an app "container-friendly," stated as a storage/networking discipline?
4. Why must image contents not encode environment specifics?

## Further Reading

- Docker networking / storage docs (concepts map to every runtime)
- CNI spec — the K8s evolution of this page

---

**← Previous:** [Registries](registries.md)
**Next:** [Why Orchestration](../kubernetes/why-orchestration.md) →
**Related:** [Linux Networking](../linux/networking.md) · [Filesystems](../linux/filesystems-permissions.md)
