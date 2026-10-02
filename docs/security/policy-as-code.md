# Policy as Code & Compliance

## What Is It?

- **Policy as code (PaC)**: expressing rules — security, cost, governance — in machine-executable form, evaluated at **admission/build time** by engines like OPA/Rego, Kyverno, Conftest, Sentinel
- **Compliance**: demonstrating conformance to frameworks (SOC 2, ISO 27001, PCI-DSS, GDPR, HIPAA) — traditionally via evidence collection and audits, now increasingly generated from the same policies and telemetry

```rego
# Kyverno: no privileged pods, ever
apiVersion: kyverno.io/v1
kind: ClusterPolicy
spec:
  validationFailureAction: Enforce
  rules:
  - name: no-privileged
    match: { resources: { kinds: [Pod] } }
    validate:
      message: "Privileged pods are forbidden"
      pattern: { spec: { containers: [["securityContext: { privileged: false }" ]] } }
```

## Why Does It Exist?

Because the previous security pages kept converging on the same sentence: *"make bad states impossible, not merely detectable."* PaC is that sentence, mechanized:

```text
Docs ("please don't X")    → ignored under pressure
Reviews ("did you X?")     → miss things, don't scale
Scans ("you X'd")          → detect after the fact
Policy ("X rejected")      → prevent at admission/build — the physics of the org
```

And on the compliance side: audits demand *evidence* ("prove S3 is never public"). PaC inverts evidence: **the rule blocks it, forever, org-wide — the proof is the policy, not a screenshot of today.**

## Layer 1 — Simple Explanation

PaC is the **building code enforced by the crane**: walls that don't meet code are never lifted into place — versus the inspector who visits after the building stands and issues fines. Compliance-as-PaC means the *city's inspection checklist maps to the crane's rules* — the audit walks the rulebook, not the tenant interviews.

## Layer 2 — Engineer's View

**The evaluation surfaces (where policies run):**

| Surface | Engine | Blocks |
|---|---|---|
| K8s admission | Kyverno / OPA-gatekeeper | insecure pods, wrong labels, missing limits |
| CI (plans/manifests) | Conftest / OPA | terraform plans with public exposure, untagged resources |
| IaC platform | Sentinel / cloud policies | disallowed SKUs, regions, services |
| Org level | SCPs / Azure Policy | the guardrails from the Cloud Security page |
| Runtime | admission-enforced signatures | unsigned images (SBOM page) |

**The policy lifecycle is software lifecycle (the whole course applies):**

```text
policies versioned in Git → tested (units per rule: "this pod must fail") →
rolled out audit-mode first (warn, measure violations) → enforce →
exceptions as pull requests with expiry → policy postmortems on incidents
```

The audit→enforce transition is the critical discipline: enforcing unmeasured rules breaks prod at 10 AM (someone's deploy IS the first violation found).

**Compliance as a continuous byproduct:**

```text
Framework (SOC 2 / PCI) → mapped to controls → controls implemented as:
   preventive policies (admission/org-level)  + detective telemetry (CSPM, audit logs)
→ compliance evidence generated continuously: policy repo + logs = the audit trail
→ audit = exporting the evidence, not assembling it by hand each quarter
```

The key mental shift: compliance requirements are **requirements on the system** — traceable, testable, versioned (the Requirements page's discipline, applied to governance). "CC6.1: access reviewed quarterly" becomes a query, not a spreadsheet.

**Where PaC strains:** highly contextual judgments (this *specific* migration legitimately needs privileged mode) — hence *exception workflows*: time-boxed, approved, visible. The exception log is itself compliance gold (and its abuse is the audit finding).

## Real-World Example (DevOps flavored)

ShopEasy's PCI scope, as policy:

```text
Frameworks: PCI-DSS for payments namespace; SOC2 org-wide
Controls-as-code:
  Kyverno:   PSS restricted enforced in payments ns; images signed+scanned only
  Conftest:  terraform plan must show encryption, no public, allowed regions
  SCP:       payments accounts denied internet gateways entirely
Evidence:    policy repo (Git history) + admission logs + CSPM reports → auditor export
Audit 2026:  evidence assembly took an afternoon, not a quarter — the byproduct design
```

## Common Mistakes

- Enforcing before auditing (breaking prod to prove vigilance)
- Policies without tests — the rule engine is code; rule bugs are prod bugs
- Policy sprawl across 4 engines with no inventory — a policy *registry* and review cadence
- No exception workflow — legitimate needs route around you (the shadow door)
- Compliance as a quarterly scramble instead of a continuous byproduct
- Writing policies that restate what org-level guardrails already enforce (layers duplicated = drift between layers)

## Mental Model

> Policy as code moves governance from the **inspector with a clipboard** to the **crane that refuses to lift a non-conforming wall**: rules versioned in Git, tested like code, enforced at the moment of creation. Compliance stops being archaeology — the audit walks your rulebook and logs, because the building literally could not have been built wrong.

## Remember This

1. PaC = prevention mechanized: admission/CI/org guardrails replace docs-review-detect chains
2. Lifecycle = software lifecycle: Git, tests, audit-mode-then-enforce, expiring exceptions
3. Surfaces: admission (Kyverno/OPA), CI (Conftest), org (SCP/Azure Policy)
4. Compliance as byproduct: preventive policies + detective telemetry = continuous evidence
5. Frameworks map to controls map to code — requirements discipline applied to governance
6. Exception workflow is the pressure valve that keeps the whole system honest

## One Sentence

Policy as code expresses organizational rules in executable form enforced at build and admission time, turning both security guardrails and compliance evidence into continuously generated byproducts of the delivery system rather than periodic human efforts.

## Knowledge Check

1. Trace "S3 never public" through docs→review→scan→policy — cost and failure mode of each.
2. Why is audit-mode-first non-negotiable? What breaks otherwise?
3. Map two SOC 2 controls to preventive policies + detective telemetry.
4. Design the exception workflow so it's a pressure valve, not a bypass.

## Further Reading

- [Open Policy Agent](https://www.openpolicyagent.org/) · [Kyverno docs](https://kyverno.io/)
- Compliance-as-code: OSCAL (NIST) — machine-readable control catalogs
- Phase complete → [Observability](../sre/observability.md)

---

**← Previous:** [Threat Modeling](threat-modeling.md)
**Next:** [Monitoring vs Observability](../sre/observability.md) →
**Related:** [Cloud Security](cloud-security.md) · [K8s & Container Security](k8s-container-security.md)
