# systemd

## What Is It?

**PID 1** on every mainstream Linux distro since ~2015: the first userspace process the kernel starts, which then starts, supervises, and cleans up everything else. A unit-based init system — and more (timers, logging via journald, mounts, network config).

The services you deploy on VMs are systemd **units**; the containers you run are units or children of units. Whatever your feelings about its scope, it is the supervisor of everything.

## Why Does It Exist?

Old init (SysV) ran startup scripts sequentially and then went to sleep — no supervision:

- Service crashes at 3 AM → stays dead until morning (no restart)
- Zombie processes accumulate (nothing reaping — recall the Processes page)
- Dependencies expressed as shell-script ordering voodoo

systemd's contract: **declare what should run and when; I'll start it correctly, restart it when it dies, watch it while it lives, and clean up after it.** (A declarative desired-state supervisor — notice the pattern that returns in Kubernetes, with reconciliation added.)

## Layer 1 — Simple Explanation

systemd is the **building manager**: opens the building (boot), starts utilities in dependency order (water before the coffee machines), patrols continuously (restart tripped breakers — crashed services), schedules maintenance (timers), keeps the incident log (journald), and locks up cleanly at night (shutdown orchestration).

## Layer 2 — Engineer's View

**The unit file — declaring a service properly:**

```ini
# /etc/systemd/system/shopeasy-api.service
[Unit]
Description=ShopEasy API
After=network-online.target       # ordering
Requires=network-online.target    # dependency semantics

[Service]
User=app
ExecStart=/opt/shopeasy/bin/api --config /etc/shopeasy/api.yml
Restart=on-failure                # the supervision you used to script by hand
RestartSec=5
LimitNOFILE=65536                 # ulimits as config
MemoryMax=1G                      # cgroup enforcement! (bridges to cgroups page)
Environment=JAVA_OPTS=-Xmx700m

[Install]
WantedBy=multi-user.target
```

This file *is* configuration management for a process — and note `MemoryMax`: systemd units are cgroup slices. Unit → slice → `system.slice` → hierarchy. `systemctl status` showing memory for the service is cgroup accounting.

**The operator's toolkit:**

| Command | For |
|---|---|
| `systemctl status/restart/mask unit` | control; `mask` = can't even be started manually |
| `journalctl -u unit -f --since "10 min ago"` | logs for the unit (next page) |
| `systemctl list-units --failed` | what died |
| `systemctl analyze blame` | boot-time hogs |
| `systemd-analyze security unit` | **a built-in per-service hardening audit** — seriously underused |

**Hardening directives worth knowing (they map directly to container security later):**

```ini
NoNewPrivileges=yes
ProtectSystem=strict        # whole FS read-only except WorkingDirectory
PrivateTmp=yes              # private /tmp — a namespace trick!
ProtectHome=yes
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

Systemd was doing "container-lite" isolation via namespaces before it was fashionable.

**Timers — cron's successor:**

```ini
# backup.timer — calendar events with logging, jitter, and missed-run handling
OnCalendar=*-*-* 02:00:00
Persistent=true    # run missed jobs after downtime — cron can't do this
```

Logs go to journald (cron emails to /dev/null), dependencies on units/services work, random delay spreads load.

**The cgroup slice tree — resource governance across the box:**

```text
-.slice
├── system.slice      ← your services, each with MemoryMax/CPUQuota
├── user.slice        ← user sessions
└── machine.slice     ← containers/VMs (Docker/K8s register here)
```

On a Kubernetes node, the kubelet configures cgroups through this same tree (QoS classes → slices). systemd isn't "old VM tech" — it's the substrate under your cluster nodes (`cgroupDriver: systemd`).

## Real-World Example (DevOps flavored)

Deploying a service on a VM the professional way (instead of a `nohup` screen session):

```bash
sudo systemctl daemon-reload          # after editing unit files — the forgotten step
sudo systemctl enable --now shopeasy-api
journalctl -u shopeasy-api -f         # watch it start
systemd-analyze security shopeasy-api # hardening score: 9.2 → tighten directives
```

And the K8s bridge: node drains (services TERM → grace → KILL), kubelet cgroup driver alignment (`systemd` vs `cgroupfs` — the classic misconfiguration causing node instability).

## Common Mistakes

- Editing unit files without `daemon-reload` — your edits silently don't apply
- `Restart=always` on a service that dies from a config error: restart storm (add `StartLimitBurst`/`StartLimitIntervalSec`)
- nohup/screen as a process supervisor — no restart, no logs, no limits
- Remaining cron-only scheduling: no logs, no missed-run handling, no jitter
- Ignoring `systemd-analyze security` when hardening VMs — free, excellent audit

## Mental Model

> systemd is the **building manager who never sleeps**: executes the startup checklist in order, restarts tripped services, records everything in the building log (journald), enforces per-suite utility budgets (cgroups), and can even lock individual suites down (namespaces) — the same job Kubernetes does for containers, done for processes.

## Remember This

1. PID 1: starts, supervises, restarts, reaps — the supervisor of everything userspace
2. Unit files are declarative process config: restart policy, limits, deps
3. `MemoryMax`/`CPUQuota` = systemd units are cgroups; slices organize the tree
4. journald + journalctl: structured, queryable logs per unit
5. Timers supersede cron: logging, persistence, jitter
6. Hardening directives prefigure container SecurityContext — same kernel features

## One Sentence

systemd is Linux's declarative supervisor — as PID 1 it starts everything in dependency order, keeps services alive via restart policies, governs their resources through cgroups, and logs them through journald.

## Knowledge Check

1. You edited a unit file; the changes don't apply. What did you forget?
2. How does `Restart=on-failure` differ from a Kubernetes liveness probe, conceptually?
3. What does `MemoryMax=1G` in a unit actually enforce, and via what kernel mechanism?
4. Name two things systemd timers do that cron cannot.

## Further Reading

- `man systemd.unit`, `man systemd.service`, `man systemd.exec`
- [systemd by example](https://systemd-by-example.com/)
- [Freedesktop — systemd docs](https://www.freedesktop.org/wiki/Software/systemd/)

---

**← Previous:** [Linux Networking](networking.md)
**Next:** [Logs & Troubleshooting](logs-troubleshooting.md) →
**Related:** [cgroups](cgroups.md) · [systemd](systemd.md) · [Observability](../sre/observability.md)
