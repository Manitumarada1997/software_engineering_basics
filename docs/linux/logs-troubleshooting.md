# Logs & Troubleshooting

## What Is It?

- **journald** — systemd's binary, structured, indexed log store (the default system log)
- Plus the classic text logs (`/var/log/...`), and — more importantly — the **methodology** of troubleshooting Linux systems from first principles

## Why Does It Matter?

This page is the payoff of the whole Linux phase: processes, memory, filesystems, network, systemd — assembled into a repeatable diagnostic method. "The server is slow" is an engineering question with a kernel-level answer.

## Layer 1 — journald

```bash
journalctl -u shopeasy-api -f                # follow one service
journalctl --since "2026-08-22 14:00" --until "14:30"
journalctl -p err -b                         # errors since boot
journalctl -t kernel | grep -i oom           # the OOM killer's own log
journalctl _PID=4321                         # everything one process said
```

Why binary + structured: filter by unit, time, priority, PID, or *custom fields* without grep-ing gigabytes; verification (tamper-evident forwarding); persistence toggled by `/etc/systemd/journald.conf` (`Storage=persistent` — on many distros logs vanish on reboot by default!).

## Layer 2 — The Troubleshooting Method

**The USE method (Brendan Gregg) — check every resource for saturation, utilization, errors:**

```bash
uptime                    # load average vs cores: 24 on 8 cores = saturation
vmstat 1                  # si/so (swap), r (run queue), wa (IO wait)
free -m                   # available memory (Memory page)
df -h && df -i            # space AND inodes (Filesystems page)
iostat -xz 1              # per-device: %util, await (disk saturation)
ss -s                     # connection counts; ss -tlnp (listening)
dmesg -T | tail -50       # kernel events: OOM kills, disk errors, NIC resets
```

**"The server is slow" — the decision tree:**

```text
High load?
├── CPU-bound?        top → us% high → which process → thread dump / profiler
├── IO-bound?         wa% high / iostat await → disk saturation or (see memory) thrashing
├── Memory-bound?     si/so storms, major faults → swap death / OOM history in dmesg
└── Not actually slow? → network/DNS (Networking page triage)
```

**The deeper instruments (know they exist, deploy when needed):**

| Tool | Layer it sees |
|---|---|
| `strace -p` / `-f` | Syscalls of a process (what is it *waiting* on?) |
| `lsof -p` | Open files/sockets (fd leaks) |
| `perf top` / FlameGraphs | On-CPU functions |
| `iotop`, `bpftool` | IO per process; eBPF-era tracing |
| `cat /proc/<pid>/stack` | Kernel-side wait point |

**`/proc` — the confession booth:** every process's memory maps, fds, limits, environ, and status are readable as files. When tooling fails, `/proc` never lies.

**The methodology principles:**

1. **Measure before restarting.** A restart destroys the evidence (heap, fds, stack state) — it converts an incident into a mystery
2. **Correlate timelines** — deploy time vs. degradation onset (CI/CD is your time-series metadata)
3. **One variable at a time**; capture, hypothesize, test
4. **Escalate depth, not guesses**: dashboards → `top/vmstat/free` → per-process (`strace`, `/proc`) → kernel (`perf`, eBPF)

## Real-World Example (DevOps flavored)

```bash
# Ticket: "staging box degraded since 13:40"
uptime && vmstat 1 && free -m      # load 38/8 cores, si/so heavy, available 80M
dmesg -T | grep -i -E "oom|kill"  # 13:41: Killed process 4182 (java) oom_score_adj
journalctl -u shopeasy-api --since 13:30 | tail   # restarts every 90s since
systemctl show shopeasy-api -p MemoryMax          # MemoryMax unset → ate the node
# fix: MemoryMax=1G in unit (+ JVM RAMPercentage), redeploy via CM, document
```

Twenty minutes, no guesses: resource → saturation → owner → constraint → fix — the USE method walking.

## Common Mistakes

- Restarting first, asking questions never — the anti-method
- Reading `free`'s wrong column, missing inode exhaustion, forgetting `dmesg` — all covered; now they're habits
- Unstructured app logs: grep-able strings instead of key=value/JSON — costs hours per incident (SRE phase fixes this properly)
- No journald persistence — reboot erases the crime scene
- Skipping the timeline correlation with deploys/config changes

## Mental Model

> A Linux box in trouble is a **patient in the ER**: journald and `/var/log` are the medical history, `top/vmstat/iostat` are the vital signs, `/proc` is the X-ray, and strace/perf/eBPF are the MRI. Restarting is defibrillation before diagnosis — sometimes necessary, always information-destroying.

## Remember This

1. journalctl: filter by unit/time/priority; make it persistent
2. USE method: every resource — utilization, saturation, errors — in a fixed order
3. `uptime, vmstat, free, df -i, iostat, ss, dmesg` = the first-response kit
4. `/proc` never lies; strace tells you what a process waits on
5. Measure before restart; correlate with deploy/config timelines
6. Go deeper (strace → perf → eBPF), not wider (guesses)

## One Sentence

Linux troubleshooting is the discipline of reading the machine's own evidence — journald, /proc, and resource metrics — through a fixed method (USE) instead of restarting and hoping.

## Knowledge Check

1. Load is 50 on an 8-core box with CPU at 20% — where is the load coming from?
2. Why is "restart it" an incident-response failure?
3. What three journalctl invocations would you use for "errors from the payments service around 2 AM"?
4. Walk the USE method for a suspected disk bottleneck.

## Further Reading

- Brendan Gregg — [The USE Method](https://www.brendangregg.com/usemethod.html) and *Systems Performance* (the bible)
- Julia Evans — zines on strace, /proc, debugging
- `man journalctl`

---

**← Previous:** [systemd](systemd.md)
**Next:** [Namespaces](namespaces.md) →
**Related:** [Logs → Observability](../sre/logging.md) · [Processes & Threads](processes-threads.md)
