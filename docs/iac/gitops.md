# GitOps

## What Is It?

**GitOps** (term from Weaveworks, 2017): operating systems where **Git is the single source of truth**, and **an in-cluster agent continuously reconciles the cluster to Git**.

The four OpenGitOps principles: **declarative** · **versioned & immutable** · **pulled automatically** · **continuously reconciled**.

```text
Push model (classic CI/CD):  CI has creds, pushes INTO cluster     (apply from outside)
Pull model (GitOps):         cluster pulls FROM Git via agent      (ArgoCD / Flux)

Git holds desired state → agent watches → diffs cluster → syncs → reports drift as a UI badge
```

## Why Does It Exist?

Three chronic problems of push-based deployment, solved at once:

1. **Credentials**: CI holding cluster-admin kubeconfigs = your pipeline is the crown jewel to compromise. Pull model: the cluster's agent fetches Git (read) — *no cluster creds exist outside the cluster*
2. **Drift blindness**: push pipelines don't know about post-deploy change (Helm's blind spot). GitOps *reconciles continuously* — drift becomes visible and self-healing
3. **No single source of truth**: was it the pipeline? The chart? Someone's `kubectl edit`? GitOps: the answer is always "whatever the repo says"

It's the reconciliation loop's final costume: **the Git repo as etcd-of-record, ArgoCD as the controller** (Architecture page's pattern, pointed at Git).

## Layer 1 — Simple Explanation

Classic CI/CD is a **postal service**: packages (manifests) shipped *to* the house — the courier holds keys to every house, and nobody knows if someone later rearranged the furniture.

GitOps is a **house that reads the architect's site itself**: the house continuously compares itself to the posted blueprint, re-arranges its own furniture to match, and files a complaint (drift alert) if someone moves a chair by hand. The architect's studio never needs house keys.

## Layer 2 — Engineer's View

**The ArgoCD Application (the core object — a CRD, per the Helm page):**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  source: { repoURL: git@...:envs/prod, path: payments, targetRevision: main }
  destination: { server: https://k8s..., namespace: payments }
  syncPolicy:
    automated: { prune: true, selfHeal: true }   # continuous reconciliation
    syncOptions: ["CreateNamespace=true"]
```

- **The promotion becomes a Git operation**: dev → staging → prod = merge PRs between environment dirs/branches. Deployment audit = `git log` (traceability chain, completed)
- **Rollback = git revert** — seconds, reviewable, auditable
- **App-of-apps pattern**: one root Application that manages the others — the fleet's table of contents
- **Progressive delivery**: Argo Rollouts integration — Git as source of truth *and* canary gating (CD page's strategies, now Git-declared)

**The honest engineering decisions:**

| Decision | Options & trade |
|---|---|
| Env structure | branch-per-env (familiar, merge friction) vs dir-per-env (explicit, copy overhead) |
| Secrets | NEVER in Git — External Secrets Operator bridges vault→cluster (Secrets page) |
| Full vs bounded sync | prune removes out-of-Git objects — selfHeal fights legitimate hotfixes unless break-glass labels exist |
| Image updates | image-updater automation (commit new tags) vs PR-gated — automation speed vs review |
| CI's new role | CI builds/tests/signs images; **CD moves entirely to Git + agent** — the cleanest separation in the industry |

**Boundaries — what GitOps doesn't do:** cluster *creation* (Terraform's job — infra vs workload split), day-2 stateful ops (operators'), and it manages K8s-shaped state best (crossplane extends the pattern outward — watch it).

## Real-World Example (DevOps flavored)

ShopEasy's final delivery flow — the whole course in one paragraph:

```text
Dev merges PR → CI: build+test+scan+sign → image (immutable tag) → registry
              → CI commits new tag to envs/staging/payments → ArgoCD syncs (1 min)
Staging verification → merge staging→prod PR (review + approvals in the PR!)
              → ArgoCD syncs prod via canary Rollout → SLO gates → 100%
Incident: revert PR → seconds-level rollback; drift: anyone's kubectl edit
              → selfHeal reverts + audit trail shows who/what/when
```

Note what disappeared: deploy buttons, pipeline cluster-credentials, "which chart is where" archaeology, and rollback fear.

## Common Mistakes

- Secrets committed "temporarily" — now the source of truth leaks
- selfHeal without break-glass labels — legitimate emergency edits instantly reverted mid-incident
- Auto-sync everything day one — start manual-sync-with-badges, graduate to automated
- Env repos as wildlands — no app-of-apps, 200 unowned Applications
- Expecting GitOps to manage what Terraform manages (reconciler wars — Drift page)
- Deleting Git history/envs casually — Git *is* production now; protect it accordingly (branch protection, signed commits — Supply chain page)

## Mental Model

> GitOps turns the cluster into a **house that continuously reads the architect's posted blueprint and fixes itself to match** — furniture (pods), locks (policies), extensions (apps). Deployments become *commits*, rollbacks become *reverts*, and the courier's ring of master keys is retired forever.

## Remember This

1. Git = single source of truth; in-cluster agent pulls and continuously reconciles (ArgoCD/Flux)
2. Kills the credential problem (no cluster creds in CI), drift blindness, and truth ambiguity
3. Promotion = PRs; rollback = git revert; audit = git log — the traceability chain completed
4. CI's role shrinks to building artifacts; CD becomes a Git operation
5. Secrets never in Git (ESO bridge); break-glass labels for selfHeal exceptions
6. Works for K8s-shaped state today; Crossplane extends the pattern to cloud infra

## One Sentence

GitOps makes Git the single, continuously-enforced source of truth for cluster state by having an in-cluster agent pull and reconcile it — turning deployments, rollbacks, and audits into version-controlled Git operations.

## Knowledge Check

1. How does the pull model structurally eliminate the "CI holds cluster-admin" attack surface?
2. Rollback time: GitOps revert vs pipeline re-run — why the order-of-magnitude difference?
3. What breaks when selfHeal meets a legitimate 3 AM hotfix, and what's the designed escape?
4. Split responsibilities cleanly: Terraform vs ArgoCD vs operators vs CI.

## Further Reading

- [OpenGitOps principles](https://opengitops.dev) · [argo-cd.readthedocs.io](https://argo-cd.readthedocs.io/)
- "Operations by Pull Request" — the original Weaveworks post

---

**← Previous:** [Drift & Infrastructure Lifecycle](drift-lifecycle.md)
**Next:** [CIA Triad & AuthN/AuthZ](../security/cia-authn-authz.md) →
**Related:** [Kubernetes Architecture](../kubernetes/architecture.md) · [Continuous Delivery](../cicd/continuous-delivery.md)
