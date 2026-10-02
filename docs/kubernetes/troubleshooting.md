# Kubernetes Troubleshooting

## What Is It?

The diagnostic method for K8s — assembling every page of this phase (and the Linux phase beneath it) into one repeatable flow. Nothing new here; that's the point: debugging K8s is walking the same causal chain the system uses.

## The Method (memorize the flow, not the commands)

```text
1. WHAT is the object's declared state?      kubectl get/describe — spec + events
2. WHAT does the system say about reality?   events, conditions, statuses
3. WHERE in the chain did it stop?           control-plane → scheduler → kubelet → runtime → app
4. WHY does the layer below say it failed?   logs / node / /proc — drop to Linux-phase tools
```

## Layer 2 — The Triage Tree (the field guide)

**Pod not starting — read the status like a dashboard:**

| Status | Meaning | Next command |
|---|---|---|
| `Pending` | scheduler can't/won't place | `describe pod` → Events (resources/taints/affinity/PVC) |
| `ContainerCreating` (stuck) | kubelet/runtime | image pull? volume attach? CNI? node events |
| `ImagePullBackOff` | registry/auth/typo | check image name, pull secret, registry status |
| `CrashLoopBackOff` | app dies on boot | `logs --previous` (crash reason), config/secrets/deps |
| `Evicted` | node pressure | `describe node` — memory/disk pressure; QoS ranking |
| `0/1 Running` not Ready | readiness failing | probe path/port/dep — the probe IS the message |

**`kubectl describe pod` events — the system's own narrative:** read chronologically; the *last* failure before silence is your thread. (Scheduling rejections, probe failures, pull errors each name themselves.)

**Service unreachable — the 4-step descent:**

```bash
kubectl get endpointslices -l kubernetes.io/service-name=payments
# empty? → selector mismatch OR pods not Ready (Pods page) — 90% of cases
# populated? → curl the VIP from a debug pod → works? kube-proxy/node issue
kubectl run tmp --rm -it --image=nicolaka/netshoot -- bash   # dig, curl, tcpdump inside
# still stuck? drop to the node: conntrack, iptables-save | grep -i pay, ipvsadm
```

**Node problems — NotReady, pressure, evictions:**

```bash
kubectl describe node | sed -n '/Conditions/,$p'   # Pressure conditions, allocated resources
kubectl get events --field-selector involvedObject.kind=Node
# on the node — the Linux page's USE method verbatim:
df -h /var/lib/containerd    # imagefs disk (eviction trigger!)
dmesg | grep -i oom ; free -m ; uptime
journalctl -u kubelet -f     # the agent's own confession
systemctl status containerd
```

**App misbehaving but "Running":** logs (`-f`, `--previous`, `--since`), `kubectl exec` (netshoot image), resource usage `kubectl top pod` vs requests (throttle check — cgroups page), then **host-level `/proc` for ground truth** (`nsenter` — your Linux-phase flex when exec fails).

**Control-plane suspicion** ("kubectl hangs", changes don't propagate): api-server healthz, etcd quorum/disk (Architecture page), RBAC "forbidden" vs "not found" (`kubectl auth can-i`).

## The Principles Behind the Commands

1. **Declare → observe → converge is the system's own debug protocol.** Your debugging mimics the reconciler: compare desired (spec) to observed (status/events), find the layer where convergence stopped
2. **Blast-radius narrowing:** cluster → node → pod → container → process → syscall. Each level has its native instrument
3. **Events before intuition** — the system narrates its failures; read the transcript before theorizing
4. **The bottom is always Linux.** Every K8s mystery terminates in something from the Linux phase: a cgroup OOM, an iptables drop, an unready veth, a DNS timeout, a disk-full imagefs

## Real-World Example (DevOps flavored)

One incident, full method — "payments 503s since 14:00":

```bash
kubectl get pods -l app=payments        # 4 pods, 1 Evicted, 1 Running(0/1)
kubectl describe node node-3 | grep -A3 Pressure     # MemoryPressure True
kubectl top pods -n payments --sort-by=memory        # leaked pod at 3.1Gi (limit 1.5? no — BestEffort!)
# root cause: new deploy dropped requests/limits → BestEffort → eviction roulette
# near-term: rollout restart; real fix: LimitRange defaults + admission policy
dmesg | grep -i "killed process"        # confirm the kernel's own account — chain closed
```

Fifteen minutes, root cause, policy fix — no guessing: status → events → node truth → kernel log.

## Common Mistakes

- Debugging the app before checking the pod's phase/events (half of incidents end at `describe`)
- Reading logs of a pod that isn't Running yet (ContainerCreating has no logs — read events)
- Missing `--previous` on CrashLoop logs — the crash's last words
- Fighting symptoms (restarts) instead of the pressure condition (evictions have causes)
- Forgetting the host view (`/proc`, nsenter) when the runtime is too sick for exec

## Mental Model

> Debugging K8s is **following the assembly line with a clipboard**: read the work order (spec), find the station where the product stopped (Pending/CrashLoop/NotReady), and interview that station's operator (events, logs, node conditions). The factory floor is Linux — and you already speak Linux.

## Remember This

1. Method: spec → events/status → identify stopped layer → drop one level deeper
2. Pod phase table: Pending→scheduler, ContainerCreating→kubelet/runtime, CrashLoop→app boot, 0/1→probe
3. Service triage: Endpoints empty = selector/readiness — the 90% case
4. Node: Pressure conditions + imagefs disk + kubelet journal + dmesg OOM
5. Events before theories; the bottom layer is always the Linux phase

## One Sentence

Kubernetes troubleshooting is tracing the same declare-observe-converge chain the system runs — from spec through events and node conditions down to Linux ground truth — with each pod status naming the exact layer where convergence stopped.

## Knowledge Check

1. A pod is Pending for 10 minutes. What are the four possible families of cause and their evidence?
2. Service "connection refused" — walk the descent to an answer.
3. Why does "Running" not mean "serving"? Which two objects bridge that gap?
4. Your cluster's pods randomly Evict at night. Give a three-hypothesis list with the test for each.

## Further Reading

- [kubectl Cheat Sheet — kubernetes.io](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- *Kubernetes Troubleshooting* — natelandau.com series; netshoot (the swiss-army debug image)
- Your own Linux phase — the last mile is always there

---

**← Previous:** [Helm & Operators](helm-operators.md)
**Next:** [IaC Fundamentals](../iac/iac-fundamentals.md) →
**Related:** [Logs & Troubleshooting (Linux)](../linux/logs-troubleshooting.md) · [Observability](../sre/observability.md)
