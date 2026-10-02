# Memory Management

## What Is It?

How Linux gives every process the illusion of its own private, contiguous memory — **virtual memory** — and manages the physical reality behind it.

```text
Virtual address (what the process sees)
    ↓ page tables (MMU hardware)
Physical frames + swap (what's really there)
```

## Why Does It Matter to a DevOps Engineer?

Because the most common production killer — **OOM (out of memory)** — is decided by these mechanics:

- Why a container gets OOMKilled at "only 500 MB" while the JVM reports 300 MB used
- Why Linux uses "all" your RAM and that's *good* (page cache)
- Why swapping is either fine (rare cold pages) or catastrophic (heap in swap)
- Every `free`, `top`, OOMKilled event, and JVM `-Xmx` decision is this page

## Layer 1 — Simple Explanation

Each process believes it owns a huge private warehouse (virtual address space). The kernel actually keeps a map: which shelf of the shared real warehouse (RAM) corresponds to each of your shelves — and only materializes a shelf **when you first touch it**.

Three consequences:

1. **Overcommit:** the kernel promises more warehouses than physical shelves exist (most programs never touch what they requested)
2. **Demand paging:** memory materializes on first touch (a `malloc` is a promise, not a delivery)
3. **Page cache:** the kernel fills spare RAM with cached disk reads — free RAM is *wasted* RAM

## Layer 2 — Engineer's View

**Reading `free -m` correctly (the classic misread):**

```text
              total   used   free   shared  buff/cache   available
Mem:           7.8G   4.1G   200M    120M       3.5G        3.4G
```

- `free` 200M looks scary; **`available` 3.4G is the real number** (includes reclaimable cache)
- "Linux ate my RAM" is the page cache doing its job: read a 2GB file twice, second read is RAM-speed

**The OOM killer — how the kernel chooses a victim:**

When memory + swap are exhausted, the kernel scores processes (`/proc/<pid>/oom_score`: mostly proportional to memory used) and kills the highest. In Kubernetes, the cgroup limit makes this deterministic: **the container exceeding its cgroup memory limit gets OOMKilled** (`OOMKilled` exit code 137 = 128+9).

**The memory-accounting trap every JVM/K8s engineer hits:**

```text
Container limit: 512 MB
JVM heap -Xmx:  400 MB  ← "fits"
Reality: heap + metaspace + thread stacks + code cache + direct buffers + page cache? > 512 MB
Result: OOMKilled at "300 MB used" (heap metric), the rest invisible to the app's dashboard
```

Fix: size `-XX:MaxRAMPercentage=50-75` against the *container* limit, not heap guesswork; watch `container_memory_working_set_bytes`, not just heap.

**Swap — nuanced, not evil:**

- Kernel swaps cold anonymous pages to make room for hot page cache = healthy
- A latency-sensitive heap swapped = death by a million microsecond-cuts (`si/so` columns in `vmstat`)
- Modern K8s default: swap off; workloads that need it (rare) set it deliberately

**Page faults — the metric under the latency:**

- Minor fault: page exists, just unmapped (fast)
- Major fault: must hit **disk** (`ps -o majflt`; `sar -B`) — major faults/sec is an IO-disguised-as-CPU problem; containers with low memory limit thrash on major faults while "CPU" looks fine

**Copy-on-write (COW):** fork() shares pages; copies happen only on write. This is how fork-based servers (and Redis BGSAVE!) clone gigabytes cheaply — until the writes come.

## Real-World Example (DevOps flavored)

The recurring incident, decoded:

```bash
kubectl describe pod payments-7d9f    # Last state: OOMKilled, exit 137
kubectl top pod payments-7d9f         # 498Mi / 512Mi limit — right at the ceiling
# on the node:
dmesg | grep -i "killed process"      # kernel's own OOM log with the score
free -m && vmstat 1                    # available? si/so swap storms?
cat /sys/fs/cgroup/.../memory.current # cgroup's view, not the JVM's
```

Fix options ranked: right-size the limit (data first), fix the leak (heap dump, `-XX:+HeapDumpOnOutOfMemoryError`), or fix the allocation pattern (5000 thread stacks). Not: add a node — leaks grow to fill whatever you provision.

## Common Mistakes

- Reading `free`'s `free` column instead of `available`
- Sizing JVM heap in isolation from the container limit (the invisible non-heap)
- Treating swap as always-bad or always-fine — it depends on what pages and why
- OOMKilled → "increase the limit" reflex without measuring what's actually counted
- Ignoring page cache when benchmarking "disk" performance (you measured RAM)

## Mental Model

> Virtual memory is a **bank issuing more loans than it has cash** (overcommit), assuming not everyone withdraws at once — with a bouncer (OOM killer) who ejects the biggest borrower when the vault empties. Page cache is the bank keeping popular cash in the teller drawers: free vault space is wasted teller space.

## Remember This

1. Virtual → physical via page tables; memory materializes on first touch (demand paging)
2. `available`, not `free`; page cache makes "used RAM" a feature
3. OOMKilled = cgroup limit exceeded — includes non-heap: metaspace, stacks, direct buffers
4. Major page faults = disk IO disguised as latency
5. Swap: healthy for cold pages, fatal for hot heaps
6. COW makes fork cheap — until writes

## One Sentence

Linux virtual memory overcommits, pages on demand, and caches disk in spare RAM — and when a container exceeds its cgroup limit, the OOM killer makes the final call by kernel rules, not by your dashboard's definition of "used."

## Knowledge Check

1. JVM shows 300 MB heap used but the container OOMKilled at a 512 MB limit. Where's the rest?
2. Why is `available` the number that matters in `free`?
3. CPU looks idle but latency spikes and `sar -B` shows major faults. What's happening?
4. What does exit code 137 mean, mechanically?

## Further Reading

- `man 5 proc` (`/proc/meminfo` fields), Brendan Gregg's Linux performance pages
- [The Linux Kernel docs — memory management](https://docs.kernel.org/admin-guide/mm/)
- [JvmContainerRAM mystery — "Why is my container OOMKilled"](https://www.evanjones.ca/java-vs-container-memory.html) (Evan Jones's classic write-ups)

---

**← Previous:** [Processes & Threads](processes-threads.md)
**Next:** [Filesystems & Permissions](filesystems-permissions.md) →
**Related:** [cgroups](cgroups.md) · [Logs & Troubleshooting](logs-troubleshooting.md)
