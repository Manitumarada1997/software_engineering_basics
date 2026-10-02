# Zero Trust

## What Is It?

A security architecture (NIST SP 800-207, ~2019-2020 formalization; Google's BeyondCorp was the pioneer, ~2011) built on one refusal:

> **Network location confers no trust.** Every request is authenticated, authorized, and encrypted — regardless of whether it comes from the office LAN or the public internet.

```text
Castle-and-moat:      hard outer wall, soft trusted interior
Zero Trust:           no interior — every door checks every badge, every time
"Never trust, always verify" — identity is the perimeter
```

## Why Does It Exist?

Because the moat model's fatal assumption — *inside = trusted* — collapsed under three pressures:

1. **Perimeter dissolution**: cloud, remote work, SaaS — there is no "inside" anymore (Cloud phase: identity replaced perimeter as the trust model)
2. **Lateral movement**: once past the moat (phished laptop, VPN access), attackers roam freely — the soft interior *is* the breach amplifier
3. **Insider reality**: most damage is mistakes, not moles — the interior can't even trust itself

BeyondCorp's insight (post-Aurora attack): employees should work *from the internet with no VPN* — access decided per-request by identity + device posture, not network position.

## Layer 1 — Simple Explanation

The castle keeps **one drawbridge and trusts everyone inside the walls**. Zero Trust is the **city where every building, every floor, every office door checks badges** — the bad guy who breaches the lobby gets... a lobby.

The VPN reframing (Firewalls page promised this): a VPN becomes merely an *encrypted road* — privacy in transit — while *authorization* happens per-door. "Connected to the network" stops meaning anything.

## Layer 2 — Engineer's View

**The three pillars (what's actually verified per request):**

| Pillar | Verifies | Your existing pieces |
|---|---|---|
| **Identity** | who (user/service) | OIDC/SAML, MFA, short-lived workload identity |
| **Device** | health of the endpoint | posture checks, managed devices, compliance |
| **Network/policy** | context + authorization | mTLS, policy engines, per-request AuthZ |

Plus: **least privilege** (RBAC/ABAC page), **assume breach** (segmentation limits blast radius — the VPC tiering pattern at every granularity), and **continuous verification** (not login-once-roam-forever).

**The implementation stack you can already name:**

```text
Users:        Identity-aware proxies / ZTNA (beyond VPN): per-app, posture-checked access
Workloads:    mTLS everywhere (mesh: SPIFFE/SPIRE identities) + per-request AuthZ
Microsegmentation: NetworkPolicies (K8s page) + cloud SGs down to per-workload
Policy:       policy-as-code engines deciding per call (OPA — later page)
Signals:      device posture + risk analytics feeding decisions
```

**The engineering honesty — what Zero Trust is NOT:**

- Not a product you buy ("zero-trust solution" marketing ≈ moat with new paint)
- Not binary adoption — it's a maturity direction: start with the highest-value interior doors (admin planes, prod data) — the Tier-0 protection from the IAM page *is* zero-trust work
- Not "no VPN" as a slogan — it's *authorization independent of network*, which may use encrypted tunnels as transport

**The platform engineering connection (foreshadow):** Zero Trust at scale *demands* a platform — hundreds of services can't each implement identity/device/policy correctly. The mesh, the identity plane, the policy engine: golden-path infrastructure (Phase 11) is what makes Zero Trust tractable for 500 engineers.

## Real-World Example (DevOps flavored)

ShopEasy's staged Zero Trust program (the realistic version):

```text
Stage 1: kill the flat network — admin access via identity-aware proxy (no VPN-admin),
         databases reachable only via private endpoints + per-service AuthZ
Stage 2: service mesh with mTLS (SPIFFE identities) — every internal call authenticated;
         default-deny NetworkPolicies everywhere (K8s page posture)
Stage 3: per-request AuthZ via OPA sidecars; device posture on human access
Measured: an SSRF-in-one-pod now reaches... a lobby. Lateral movement graph flatlines.
```

## Common Mistakes

- Buying "Zero Trust" as a box while the flat interior remains
- Identity-only trust ("we have SSO") without segmentation — one phish still roams
- All-at-once programs stalling — it's a direction, sequenced by blast radius
- Trusting the mesh's mTLS while leaving admin planes on legacy network trust
- Ignoring device posture for BYOD-heavy orgs — the weakest pillar becomes the door

## Mental Model

> Castle-and-moat has one **drawbridge and a soft interior**; Zero Trust is a **city where every single door checks every badge every time** — the VPN demoted from gatekeeper to mere encrypted road. Breaches still happen (someone loses a badge) — but the burglar in the lobby finds... a lobby.

## Remember This

1. Core refusal: network location ≠ trust; identity+device+policy verify every request
2. Drivers: cloud/remote dissolved the perimeter; lateral movement punished the soft interior
3. Pillars: identity (OIDC/MFA), device posture, per-request AuthZ + microsegmentation
4. Implementation = pieces you know: ZTNA, mTLS/SPIFFE, NetworkPolicies, policy-as-code
5. It's a maturity direction, not a product or a project — sequence by blast radius
6. At scale it *requires* platform infrastructure — Phase 11's seed

## One Sentence

Zero Trust removes network location as a source of trust, verifying identity, device, and authorization on every request so that breaches land in a lobby instead of the crown-jewel corridor.

## Knowledge Check

1. What did the VPN *mean* in castle terms, and what does it mean after Zero Trust?
2. Map the three pillars onto tools you already operate.
3. Why does Zero Trust fail as a big-bang project? Propose a first-year sequence.
4. Why does 500-service Zero Trust imply a platform team?

## Further Reading

- NIST SP 800-207 — the canonical paper (short)
- Google BeyondCorp papers — the origin story

---

**← Previous:** [OAuth 2.0, OIDC & SAML](oauth-oidc-saml.md)
**Next:** [PKI & Encryption](pki-encryption.md) →
**Related:** [Firewalls & VPNs](../networking/firewalls-vpns.md) · [Cloud IAM](../cloud/iam.md)
