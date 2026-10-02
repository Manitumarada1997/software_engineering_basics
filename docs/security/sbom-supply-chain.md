# SBOM & Supply Chain Security

## What Is It?

- **SBOM** (Software Bill of Materials): a machine-readable **inventory of every component in an artifact** — dependencies, versions, licenses, hashes (SPDX / CycloneDX formats)
- **Supply chain security**: protecting the chain from source → build → artifact → deploy — because your software is ~80% other people's code, delivered through ~8 links you don't control

```text
your source → git → CI → build tools → dependencies → registry → deploy → runtime
     ↑SolarWinds-style attacks target exactly these links↑
```

## Why Does It Exist?

Two modern realizations:

1. **You ship what you don't know**: Log4Shell (2021) forced every org on earth to answer *"where do we run log4j 2.x?"* — orgs with SBOMs answered in minutes; without, in weeks. An inventory you generate *before* the crisis is the entire value
2. **The chain itself is attackable**: compromised build systems (SolarWinds, 2020), hijacked namespaces, typosquatting, malicious commits — attacks *upstream* of your code, inheriting all your trust

The industry's response framework: **SLSA** (Supply-chain Levels for Software Artifacts) — maturity levels for build integrity — and **Sigstore** (cosign signing) — verifiable provenance.

## Layer 1 — Simple Explanation

The SBOM is the **ingredients label** for software — every component, version, supplier. When the news says "brand X flour recalled", you don't recall every product you've ever baked: you read your labels and pull exactly the affected batches.

Supply-chain security is the **sealed-truck program**: tamper-evident packaging (signatures), verified shippers (provenance: "this artifact really came from that source, built by that trusted builder"), and a loading dock that refuses unsealed goods (admission control).

## Layer 2 — Engineer's View

**Generating and using SBOMs (the operational loop):**

```bash
# at build: generate, attach to the OCI artifact (OCI page: registries carry more than images)
syft registry/shop/payments:2.1.0 -o cyclonedx-json > sbom.cdx.json
cosign attest --predicate sbom.cdx.json registry/shop/payments:2.1.0
# at incident: "which artifacts contain lib X < Y?"
grype sbom:registry/shop/payments:2.1.0      # or query your SBOM warehouse
# at deploy: admission verifies signature + attestation before the pod runs
```

The missing-piece rule: SBOM without a *queryable store* = a filing cabinet you never open during the fire.

**SLSA levels — the maturity ladder (learn the names):**

| Level | Guarantees |
|---|---|
| 1 | build provenance documented |
| 2 | hosted build service, signed provenance |
| 3 | hardened builds — isolation, non-falsifiable provenance |
| 4 | two-party review + reproducible builds |

You can't jump to 4; you start by *making provenance exist* (CI metadata), then signing it, then hardening the builders.

**Sigstore/cosign — the practical signing stack:**

```text
sign: keyless (OIDC identity of the CI) → transparency log entry → signature on digest
verify: admission ("only digests signed by our CI principal, attested SBOM present")
```

The elegant part: **keyless signing binds artifacts to the workflow identity** — no signing keys to leak; the transparency log makes signatures public-auditable.

**The link-by-link defense checklist:**

| Link | Control |
|---|---|
| Source | signed commits, branch protection, two-person review for critical repos |
| Dependencies | lockfiles, allowlisted registries (proxy), SCA + reachability |
| Build | hermetic builds (no network), isolated builders, provenance attestation |
| Artifacts | digest-pinned, signed, SBOM-attached, immutable registry |
| Deploy | admission verifying signature+SBOM (the K8s security page's gate) |

**The GitOps connection (it all lands there):** the Git repo signed and protected = the source-of-truth anchor; ArgoCD's admission + verification = the enforcement dock. The supply chain becomes *verifiable end-to-end*: commit → build → artifact → running pod, each transition cryptographically tied.

## Real-World Example (DevOps flavored)

Log4Shell, runbook for the SBOM-era org:

```text
14:00  CVE published (log4j 2.x RCE)
14:05  query SBOM store: "artifacts containing log4j 2.x" → 14 artifacts, 6 in prod
14:20  patched dependency merged; CI rebuilds; new digests signed+SBOM'd
14:45  GitOps promotes; admission accepts only the new signed digests
15:10  verification query: zero remaining — done, documented, before most orgs' first meeting
```

The delta isn't tooling speed — it's having *answered the inventory question in advance*.

## Common Mistakes

- SBOMs generated and archived — never queryable at incident time (filing-cabinet theater)
- Trusting unsigned CI because "it's our CI" — the SolarWinds lesson is precisely this sentence
- `latest` tags and floating deps — the chain's weakest links re-introduced at the end
- Skipping dependency-registry allowlisting — typosquatting is a numbers game against inattentiveness
- Reproducibility claims never tested — verify a rebuild matches, occasionally

## Mental Model

> The SBOM is the **ingredients label on every product you ship**; supply-chain security is the **sealed-truck program** — signed provenance from field to shelf, and a loading dock that refuses anything unsealed. When the recall news breaks, the labeled, sealed operation pulls exact batches in minutes; the unlabeled one empties the whole warehouse in panic.

## Remember This

1. SBOM = component inventory (SPDX/CycloneDX) — generated at build, queryable at incident
2. SLSA = build-integrity maturity ladder; Sigstore = keyless signing bound to CI identity
3. The chain's links: source/deps/build/artifact/deploy — each has a specific control
4. Admission-enforced verification (signature+SBOM) turns policy into physics
5. Log4Shell is the case study: inventory-in-advance is the entire value
6. GitOps + signing = end-to-end verifiable delivery

## One Sentence

Supply-chain security makes every step from source to runtime cryptographically verifiable — SBOMs for knowing what you ship, SLSA/Sigstore for proving how it was built — so that both upstream attacks and recall-style incidents become bounded, queryable events.

## Knowledge Check

1. Walk your artifact's chain link-by-link; name the control (and its absence!) at each.
2. Why is keyless signing safer than a long-lived signing key?
3. Design the 15-minute Log4Shell response — what must exist *before* the CVE?
4. What does admission enforcement add that scanner output alone doesn't?

## Further Reading

- [slsa.dev](https://slsa.dev) · [sigstore.dev](https://sigstore.dev) · SBOM formats: SPDX/CycloneDX
- Next: [Vulnerability Management](vulnerability-management.md)

---

**← Previous:** [SAST, DAST & SCA](sast-dast-sca.md)
**Next:** [Vulnerability Management](vulnerability-management.md) →
**Related:** [Registries](../containers/registries.md) · [GitOps](../iac/gitops.md)
