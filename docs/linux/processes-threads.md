# Processes & Threads

## What Is It?

- A **process** is a running program: an isolated unit with its own memory space (virtual address space), file descriptors, credentials, and at least one thread.
- A **thread** is a unit of execution *inside* a process: its own stack and registers, but **sharing the process's memory, files, and credentials** with sibling threads.

```text
Process = a house (walls = memory isolation)
Thread  = a person in the house (own hands/stack, shared kitchen/files)
```

## Why Does This Matter to a DevOps Engineer?

Because every abstraction you operate **is** processes:

- A Docker container = a process (or tree) with namespaces + cgroups (Phase 3 finale)
- A Kubernetes Pod = processes sharing namespaces on a node
- JVM application performance = threads, GC, and memory — as processes seen by the OS
- Every incident: "the service is slow/dead/hung" is answered with process-level tools: `ps`, `top`, `/proc`, signals, coredumps

## Layer 1 — Simple Explanation

A **program** is a recipe on paper. A **process** is the kitchen actively cooking it — with its own room, ingredients (memory), and equipment (file descriptors). A **thread** is a cook; multiple cooks share the same kitchen, which is fast (no door between rooms) but risky (two cooks, one knife — synchronization).

## Layer 2 — Engineer's View

**The process's anatomy (what the kernel tracks per process):**

| Component | What it is | Where you see it |
|---|---|---|
| PID / PPID | Process ID; parent PID | `ps -ef`, `/proc/$$/status` |
| Virtual address space | Code, heap, stack, shared libs | `/proc/<pid>/maps` |
| File descriptors | Open files, sockets, pipes | `/proc/<pid>/fd` (yes, sockets are fds) |
| Credentials | UID/GID/groups/capabilities | `ps -o user`, capabilities → container security |
| Environment | ENV vars | `/proc/<pid>/environ` |
| Scheduling info | State, priority, CPU time | `top`, `/proc/<pid>/stat` |

**Process states (what "hung" actually means):**

```text
R running → runnable
S sleeping → waiting for an event (network read, timer) — NORMAL, not broken
D uninterruptible sleep → usually disk/IO — kills can't touch it
Z zombie → exited, parent hasn't reaped it
T stopped
```

Operational translation: a service with all threads in `D` = IO bottleneck; `S` on a socket = waiting for a downstream; `Z` accumulation = a supervisor bug (init should reap — systemd/PID 1 in containers exists largely to reap zombies).

**Signals — the process control protocol:**

| Signal | Default meaning | Ops use |
|---|---|---|
| SIGTERM (15) | Graceful stop: finish work, close connections | What `kubectl delete pod`, `docker stop` send first |
| SIGKILL (9) | Unstoppable death, no cleanup | Last resort; leaks & lost in-flight work |
| SIGHUP (1) | Terminal hangup | Convention: "reload config" for daemons |
| SIGSTOP/CONT | Pause/resume | Debugging |

The graceful-shutdown choreography you've configured in Kubernetes (`terminationGracePeriodSeconds`: TERM → wait 30s → KILL) *is* this table.

**Threads vs processes — the trade-off:**

| | Threads | Processes |
|---|---|---|
| Creation cost | cheap (~µs) | expensive (~ms) |
| Communication | shared memory (fast, needs locks) | pipes/sockets/IPC (slower, safe) |
| Failure isolation | one thread's crash kills all | isolated memory |
| The container connection | a container = process(es) | containers isolate what processes |

`/proc/<pid>/task/` exposes threads as pseudo-PIDs; `top -H` or `ps -T` shows them. Thread count matters operationally: each thread has a stack (default 1–8 MB virtual), so 5000 threads = address-space and scheduler pressure — the classic JVM/OOMKilled story.

**fork() vs exec() — how processes start (the 5-second version):** `fork()` clones the current process (copy-on-write — child shares pages until either writes), then `exec()` replaces the image with a new program. Shell `|` pipes = fork, exec, pipe fd plumbing. Zombies exist between fork and exit-reap.

## Real-World Example (DevOps flavored)

The classic production debugging session:

```bash
# "payments-api is hung"
ps -o pid,stat,wchan:30,cmd -p $(pidof java)   # all threads in D? IO. In S on futex? lock contention.
ls /proc/<pid>/fd | wc -l                      # 65,536 fds? FD leak (ulimit -n exhausted)
cat /proc/<pid>/limits                         # confirm max open files
kill -SIGQUIT <pid>                            # JVM: thread dump to stdout
```

And the container mapping that unlocks the next concepts:

```bash
docker run ... java -jar app.jar    # = one process, wrapped in namespaces + cgroups
# PID 1 problem: java as PID 1 ignores SIGTERM unless built for it → use tini/entrypoint
```

## Common Mistakes

- `kill -9` as first instinct — destroys cleanup, corrupts state, hides the real bug
- Assuming `S` (sleeping) = stuck — it usually means healthy waiting
- Ignoring PID-1 responsibilities in containers (zombie reaping, signal handling)
- Thread-per-request systems with unbounded thread creation (`OutOfMemoryError: unable to create native thread`)
- Debugging inside the container only — `/proc` on the *host* sees everything

## Mental Model

> A process is a **house with locked doors** (memory isolation); threads are its **inhabitants shouting across rooms** (shared memory, needs rules — locks). Signals are the **doorbell and the fire alarm**: TERM is "please leave," KILL is burning the house down with everyone inside.

## Remember This

1. Process = isolation (memory, fds, creds); thread = execution inside it
2. States: R, S (normal), D (IO), Z (unreaped), T — they decode "hung"
3. TERM → graceful, KILL → last resort; K8s grace periods are signal choreography
4. Everything you run is a process: containers, pods, JVMs — `/proc` is the truth
5. Threads share memory: cheap but synchronization-prone; processes: safe but IPC
6. PID 1 in containers has real jobs: signals + zombie reaping

## One Sentence

A process is an isolated unit of running program with its own memory and resources, threads are concurrent executions sharing that memory, and the kernel's signals and states are the control plane you actually operate during incidents.

## Knowledge Check

1. Your container ignores SIGTERM and always takes the 30s grace-period kill. Why?
2. All threads in `D` state vs all in `S` — what's the different diagnosis?
3. Why do 10,000 threads hurt even with plenty of RAM?
4. What is a zombie and who is supposed to prevent them in a container?

## Further Reading

- `man 7 signal`, `man 2 fork`, `man proc` — the actual documentation
- *The Linux Programming Interface* — Michael Kerrisk, ch. 2–3, 20–21 (reference-grade)
- Julia Evans, *Linux Debugging tools I love* (blog/zines)

---

**← Previous:** [Pipeline as Product](../cicd/pipeline-as-product.md)
**Next:** [Memory](memory.md) →
**Related:** [Namespaces](namespaces.md) · [cgroups](cgroups.md) · [Logs & Troubleshooting](logs-troubleshooting.md)
