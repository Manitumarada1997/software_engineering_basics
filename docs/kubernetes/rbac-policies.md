# Namespaces, RBAC & Network Policies

## What Is It?

K8s' **soft multi-tenancy** layer — three objects that partition one shared cluster:

- **Namespace**: a virtual cluster boundary (names, quotas, policies scoped within)
- **RBAC** (Role / ClusterRole + bindings): *who* may *do what* to *which API objects* — the IAM page's model, cluster edition
- **NetworkPolicy**: pod-level firewalls (the Firewalls page's default-deny, CNI-enforced)

## Why Does It Exist?

One cluster, many teams and environments — without partitioning, you get: name collisions, noisy neighbors, cluster-admin for everyone, and flat networking where any pod reaches any pod. The three objects answer: *scope* (namespace), *API authority* (RBAC), *network reachability* (NetworkPolicy).

**The security caveat that frames everything:** namespaces are *organizational*, not *security* boundaries. Strong isolation (strangers, regulated workloads) → separate clusters or vClusters; namespaces separate *colleagues*.

## Layer 1 — Simple Explanation

- **Namespace** = the **department's floor** in one building: own desks (names), budget (quota), and wall
- **RBAC** = the **badge system**: this badge opens the payments floor's cabinets (RoleBinding); facilities managers hold a building-wide badge (ClusterRole) — granted sparingly
- **NetworkPolicy** = the **internal door locks**: "pods with label `app=payments` may talk to `app=db` on 5432; nothing else"

## Layer 2 — Engineer's View

**RBAC grammar (four objects, one mental model):**

```yaml
kind: Role                      # WHAT: verbs × resources, namespaced
rules: [{ apiGroups: [""], resources: ["pods"], verbs: ["get","list"] }]
---
kind: RoleBinding              # WHO gets it, in this namespace
subjects: [{ kind: User, name: dev-team }]
roleRef: { kind: Role, name: pod-reader }
# ClusterRole / ClusterRoleBinding = the cluster-wide versions
```

Mapping to the IAM page: principal → binding → role (actions × resources). Escalation risks live at the edges — `create pods` = "act as any serviceaccount that exists" (pod spec picks SA!); `bind`/`escalate`/`impersonate` verbs and `*` resources are Tier-0 adjacent. Audit: `kubectl auth can-i --as=dev@team list secrets -n prod` — RBAC is *testable*.

**Workload identity, K8s edition:** the **ServiceAccount** is a pod's identity. Modern practice: one SA per workload, token automounting disabled where unused, short-lived projected tokens, and — the cloud bridge — **workload identity federation** (SA → cloud role without stored keys, the IAM page's OIDC pattern). CI/CD to cluster: OIDC, never long-lived kubeconfigs.

**ResourceQuota + LimitRange** — namespaces' economic walls: quota (total requests/limits/objects per ns) prevents one team's runaway job from starving the cluster; LimitRange defaults prevent "no requests set" pods (Pods page's QoS chaos).

**NetworkPolicy mechanics:**

```yaml
kind: NetworkPolicy
spec:
  podSelector: { matchLabels: { app: db } }      # applies to these pods
  policyTypes: [Ingress]
  ingress:
  - from: [{ podSelector: { matchLabels: { app: payments } } }]
    ports: [{ port: 5432 }]
```

- **Selecting a pod with ANY NetworkPolicy makes it default-deny** for that direction — the "why did my app break after adding one policy" moment
- Requires a CNI that enforces (Calico/Cilium — plain flannel doesn't!)
- The standard posture: namespace-level default-deny + explicit allows = the VPC tiering pattern (Cloud page) at pod granularity
- L7 policies (Cilium HTTP rules) extend this — an API-gateway-ish layer inside the mesh

## Real-World Example (DevOps flavored)

ShopEasy's tenant model:

```yaml
# per-team namespace template:
ResourceQuota: { requests.cpu: "20", requests.memory: 40Gi, pods: 100 }
LimitRange: defaults requests=limits (Guaranteed QoS)
NetworkPolicy: default-deny ingress/egress + allow DNS + allow labeled flows
RBAC: team group → Role (full CRUD in own ns), no cluster roles;
      platform team → break-glass ClusterRole (audited, time-boxed)
CI: OIDC token → RoleBinding deploy-role in the target ns only
```

The audit finding it prevents: a compromised app pod pivoting to read every Secret in the cluster — default-deny + namespace-scoped SAs contain the blast radius to one floor.

## Common Mistakes

- cluster-admin for CI "because permissions were complex" — the pipeline is now the cluster's crown
- One shared ServiceAccount for all workloads — identity meaningless, forensics impossible
- NetworkPolicies added without default-deny strategy — partial policies = partial confusion
- Believing namespaces = hard security isolation
- No ResourceQuota — the single runaway CronJob that ate the cluster's memory budget

## Mental Model

> A namespace is a **floor**, RBAC the **badge system** (role = clearance list, binding = who holds it), NetworkPolicy the **door locks between rooms**. One building shared by colleagues — convenient, efficient, and *soft-walled*: real strangers get their own building (separate cluster).

## Remember This

1. Namespaces = scope + quotas (soft tenancy); hard isolation needs separate clusters
2. RBAC: role (verbs×resources) + binding (subjects); testable with `kubectl auth can-i`
3. Dangerous edges: `create pods`, impersonation, wildcard rules, cluster-admin
4. ServiceAccount = workload identity; one per workload + cloud federation, no stored keys
5. Any NetworkPolicy selection = default-deny that direction; needs an enforcing CNI
6. Quotas + LimitRanges make floors economically fair

## One Sentence

Namespaces partition the cluster into scoped floors, RBAC controls who may act on which API objects within them, and NetworkPolicies enforce default-deny pod networking — together providing soft multi-tenancy whose blast-radius value depends on combining all three.

## Knowledge Check

1. Why does `create pods` in a namespace confer more power than it appears to?
2. A pod becomes unreachable after you add one NetworkPolicy. Explain the mechanism.
3. Design CI→cluster auth without any stored kubeconfig.
4. What do ResourceQuotas prevent that requests/limits per pod don't?

## Further Reading

- [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) / [NetworkPolicies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) — kubernetes.io
- Next: [Autoscaling & Scheduling](autoscaling.md)

---

**← Previous:** [StatefulSets, DaemonSets, Jobs](workloads.md)
**Next:** [Autoscaling & Scheduling](autoscaling.md) →
**Related:** [Cloud IAM](../cloud/iam.md) · [Firewalls & VPNs](../networking/firewalls-vpns.md)
