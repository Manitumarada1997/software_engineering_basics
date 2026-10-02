# Threat Modeling

## What Is It?

A **structured design review for security**: systematically asking *what can go wrong, against what, and are we okay with that* — before and during building. The four questions (Shostack, *Threat Modeling: Designing for Security*):

```text
1. What are we building?        (diagram: data flows, trust boundaries)
2. What can go wrong?          (threats — via frameworks like STRIDE)
3. What are we going to do?     (mitigations → design changes)
4. Did we do a good job?        (validation, retromanifestation)
```

It's the security chapter of architecture review — shift-left applied to *design*, before code exists.

## Why Does It Exist?

Because scanners (previous pages) find **implementation bugs in finished things** — they cannot find **design flaws**: the unauthenticated internal API "because it's internal" (Zero Trust page fiction), the admin plane reachable from the app tier, the data flow that logs PII. Design flaws are cheaper to fix on a whiteboard — the cost-of-change curve (Waterfall page), security edition.

## Layer 1 — Simple Explanation

A **fire-safety walkthrough of the blueprints** before the building rises: where are the exits (segmentation), what's flammable (crown-jewel data), who has keys (identity), what happens if this door fails (blast radius) — with the architect and the fire marshal in the same room arguing about walls, cheaply, instead of about the finished building, expensively.

## Layer 2 — Engineer's View

**STRIDE — the threat checklist per data-flow element (the standard mnemonic):**

| Letter | Threat | Violates (CIA page) |
|---|---|---|
| **S**poofing | faking identity | AuthN |
| **T**ampering | altering data/flow | Integrity |
| **R**epudiation | denying actions | non-repudiation/audit |
| **I**nformation disclosure | leaking data | Confidentiality |
| **D**enial of service | killing availability | Availability |
| **E**levation of privilege | gaining unauthorized rights | AuthZ |

The method: draw the system (boxes, data flows, **trust boundaries** dashed), then walk each flow crossing a boundary through STRIDE — systematically, element by element.

**The workflow that makes it real (not ceremony):**

```text
Trigger: new service, major design change, crown-jewel data touched
Attendees: builder + security + one curious outsider (the naive question finds the flaw)
Artefacts: data-flow diagram (living, in the repo), threat register (issue tracker)
Output: mitigations as design changes or backlog items with owners
Retrospective: at incidents — "was this in the model? why not?" (the feedback loop)
```

**The trust-boundary discipline — the single most productive habit:** most findings live exactly where data crosses a boundary (user→app, app→DB, cluster→cloud metadata). Walk boundaries hardest; interior flows matter less. (You can already draw these: CDN→gateway→service→DB — every Networking/K8s page was drawing this diagram for you.)

**PASTA / attack trees (one line each):** PASTA = risk/business-centered deep method for regulated domains; attack trees = decompose one goal ("get checkout data") into attack paths, for prioritizing defenses. STRIDE covers 80% of engineering needs; know the others exist.

**The DevOps-flavored application — model the *pipeline* too:**

```text
The delivery system is itself a crown jewel (Pipeline-as-Product page):
threats: poisoned PRs (Spoofing of contributors), tampered CI runners, leaked
         pipeline identities (IAM page), registry injection (SBOM page's chain),
         GitOps repo takeover (= prod takeover)
mitigations: signed commits, OIDC federation, isolated builders, admission
             verification, branch protection — every previous page, co-designed
```

## Real-World Example (DevOps flavored)

ShopEasy's checkout redesign, 90-minute model:

```text
Diagram: user → gateway → checkout-svc → payments-db (+ events → queue → fraud-svc)
Boundaries: internet→gateway; gateway→svc; svc→db; svc→queue
Findings (STRIDE pass):
- I: fraud-svc logs full card numbers from events → mask at emit (design change, pre-code)
- E: queue consumers run with DB-network access unnecessarily → egress deny (Network page)
- S: gateway→svc internal calls unauthenticated ("internal") → mTLS + service identity
- D: queue depth unbounded → backpressure limits (Availability as security)
Each → backlog item with owner; diagram committed to repo; revisit at incident reviews
```

Three design-level fixes, zero code written yet — the entire ROI of the method.

## Common Mistakes

- Threat modeling as annual compliance theater on stale diagrams
- Modeling without the builder present — security guessing at intent
- No trust boundaries drawn = the method neutered
- Register without owners/dates — findings as decoration
- Skipping the feedback loop (incidents never improve the model)
- Trying to enumerate all threats (impossible) instead of walking boundaries systematically

## Mental Model

> Threat modeling is a **fire marshal's walkthrough of the blueprints**: diagrams with trust boundaries as fire walls, STRIDE as the inspection checklist, mitigations as design changes while walls are still lines on paper. Scanners find faulty wiring *after* the build; the walkthrough prevents the floor plan that guarantees fires.

## Remember This

1. Four questions: build / go wrong / do / validate — a design review, not a document
2. STRIDE = six threat classes mapped to CIA violations; walk per data-flow element
3. Trust boundaries concentrate the findings — walk them hardest
4. Builder + security + outsider; register with owners; diagrams live in the repo
5. Model the delivery system itself — the pipeline is a crown jewel
6. Incident reviews feed the model — the loop that makes it durable

## One Sentence

Threat modeling is a structured design review that diagrams trust boundaries, walks them against threat checklists like STRIDE, and turns findings into design changes while change is still cheap.

## Knowledge Check

1. Walk one flow from your current system through all six STRIDE letters.
2. Why do scanners complement, never replace, this practice?
3. What makes a trust boundary the productive focus? Give three boundaries in your estate.
4. Model the threat "pipeline identity theft" end-to-end — mitigations by page of this course.

## Further Reading

- *Threat Modeling: Designing for Security* — Adam Shostack
- Microsoft STRIDE docs; [OWASP Threat Dragon](https://owasp.org/www-project-threat-dragon/) (tooling)
- Next: [Policy as Code & Compliance](policy-as-code.md)

---

**← Previous:** [Vulnerability Management](vulnerability-management.md)
**Next:** [Policy as Code & Compliance](policy-as-code.md) →
**Related:** [Threat modeling ↔ architecture review](../architecture/thinking-tradeoffs.md)
