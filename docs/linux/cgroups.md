# cgroups (Control Groups)

## What Is It?

**cgroups** are the kernel feature that **measures, limits, and prioritizes resource usage** for groups of processes: CPU, memory, IO, devices, PIDs.

If namespaces isolate what a process *sees*, cgroups ration what it *gets*:

```text
namespaces → view ("my own world")
cgroups    → resources ("my own ration")
```

## Why Does It Exist?

The noisy-neighbor problem. Multiple workloads, one machine: one greedy process can starve everyone (memory → OOM chaos, CPU → latency death). Before cgroups (2007, Google; rewritten as **cgroups v2** ~2020), the options were process priority (weak) or separate VMs (heavy).

cgroups gave the kernel per-group accounting and enforcement — enabling both containers *and* Kubernetes QoS classes ("this pod is evictable, that one is guaranteed").

## Layer 1 — Simple Explanation

cgroups are the **hotel's utility metering**: each room (group) gets measured electricity (CPU), water (memory), and bandwidth (IO). Exceed your water allocation and the consequences are contractual — throttled (CPU: slowed), reclaimed (memory: OOMKilled), or deprioritized (IO: queued last). And the meters feed a central dashboard (`/sys/fs/cgroup`) — which is how the hotel bills you (resource accounting → monitoring).

## Layer 2 — Engineer's View

**The v2 filesystem — literally readable config:**

```bash
ls /sys/fs/cgroup/                  # controllers: cpu, memory, io, pids...
# create & constrain a group:
mkdir /sys/fs/cgroup/demo
echo "max 100000 100000" > demo/cpu.max     # 1.0 CPU (quota/period)
echo 100M > demo/memory.max                 # hard limit → OOM kill on exceed
echo 500 > demo/pids.max                    # fork-bomb protection
echo $$ > demo/cgroup.procs                 # join this shell into the group
```

**The controllers that run your production:**

| Controller | Enforces | Container/K8s reality |
|---|---|---|
| `cpu.max` | CPU quota/period | `--cpus=2` / K8s CPU limits → **throttling** |
| `memory.max` | Hard memory ceiling | K8s memory limit → **OOMKilled** (137) |
| `io.max` | Device IO ceilings | rarely tuned; disk QoS |
| `pids.max` | Process/thread ceiling | anti-fork-bomb |
| `cpu.weight` | Relative share *under contention* | K8s CPU **requests** semantics |

**The two CPU models — requests vs limits (the most-misunderstood pairing in K8s):**

```text
weight (requests) = your share WHEN the machine is contended — always runnable
max (limits)      = your ceiling ALWAYS — enforced by throttling
```

CPU throttling mechanics: within each 100ms period, a pod limited to 0.5 CPU gets 50ms of runtime — then **sleeps until next period**. Symptom: mysterious, regular latency spikes with "low CPU" — check `cpu.stat`'s `nr_throttled`. (This is why many high-performance teams drop CPU limits and keep requests.)

**Memory: no throttling exists.** Exceed `memory.max` → reclaim, then OOM kill of the group's largest scorer — deterministic, unlike host-level OOM. This asymmetry (CPU bends, memory kills) explains most container resource incidents.

**Accounting — the observability dividend:** cgroups count everything: `memory.current`, `cpu.stat` (used/throttled), `io.stat`. Every container metric you've ever seen (`kubectl top`, cadvisor, node exporter) is reading cgroup accounting files. Your dashboards are `cat`s of `/sys/fs/cgroup`.

**The systemd connection (previous pages join here):** on modern nodes, systemd owns the cgroup hierarchy: `system.slice` → your services (`MemoryMax=`), `machine.slice`/`kubepods.slice` → containers. Kubelet's `cgroupDriver: systemd` aligns itself with that tree. cgroups aren't just "in" your stack — your stack is *written in* cgroups.

## Real-World Example (DevOps flavored)

Diagnosing the classic latency sawtooth:

```bash
kubectl exec pod -- cat /sys/fs/cgroup/cpu.stat
# nr_periods 48210  nr_throttled 31144   ← 65% of periods throttled!
kubectl get pod -o jsonpath='{.spec.containers[0].resources}'
# limits: cpu 500m — but the app bursts legitimately → raise limit or drop CPU limits
```

And memory forensics:

```bash
kubectl describe pod ... # OOMKilled, exit 137
cat /sys/fs/cgroup/.../memory.events   # oom_kill 3, oom_group 1
# sizing conversation follows (memory.current trend), not node-scaling
```

## Common Mistakes

- CPU limits set = throttling latency; monitored "CPU usage" never shows it — watch `nr_throttled`
- Memory limit ≠ heap size (JVM metaspace/stacks/buffers — Memory page)
- No `pids.max` on multi-tenant nodes — one fork bomb eats the node
- Expecting CPU-style "bending" from memory limits — memory kills, it doesn't throttle
- Ignoring cgroup v2 vs v1 differences when reading node metrics (v2 unified hierarchy)

## Mental Model

> Namespaces hand each process a **VR world**; cgroups strap on the **fitness tracker with a taser**: everything measured (accounting), some things limited with a shock (throttling), and one limit that ends you (memory OOM). Kubernetes QoS classes are just pre-written taser settings.

## Remember This

1. cgroups limit & account: cpu, memory, io, pids — per process group
2. CPU throttles (quota/period); memory kills — the asymmetry behind most incidents
3. `weight` = share under contention (requests); `max` = ceiling always (limits)
4. All container/pod metrics are cgroup accounting files in `/sys/fs/cgroup`
5. systemd owns the cgroup tree on nodes; kubelet aligns (`cgroupDriver`)
6. `nr_throttled` is the metric "low CPU but slow" asks for

## One Sentence

cgroups are the kernel's per-group resource rations and meters — CPU that throttles, memory that kills, IO and process counts that cap — and every container limit, QoS class, and usage metric you operate is this mechanism wearing a Kubernetes costume.

## Knowledge Check

1. Why does "CPU at 40%" coexist with p99 latency spikes? Name the file that proves it.
2. Why do memory limits fail differently than CPU limits, mechanically?
3. Map K8s requests/limits onto `cpu.weight`/`cpu.max` semantics.
4. Where does `kubectl top` get its numbers, physically?

## Further Reading

- `man 7 cgroups`, kernel docs — [cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html)
- [Julia Evans' cgroup zines](https://wizardzines.com/)
- Next: [build a container by hand](../projects/lab-container-by-hand.md)

---

**← Previous:** [Namespaces](namespaces.md)
**Next:** [Lab: Build a Container by Hand](../projects/lab-container-by-hand.md) →
**Related:** [Memory](memory.md) · [systemd](systemd.md)
