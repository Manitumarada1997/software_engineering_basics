# Versioning & Semantic Versioning

## What Is It?

Versioning is the practice of giving identifiable, comparable names to states of software. **Semantic Versioning (SemVer)** is the standard scheme:

```text
MAJOR.MINOR.PATCH
  1 .  4 .  2

MAJOR — incompatible API changes
MINOR — new functionality, backwards compatible
PATCH — backwards compatible bug fixes
```

## Why Does It Exist?

Because software never exists alone — it exists in **dependency graphs**. Your service depends on libraries; your artifact is consumed by pipelines; your API is consumed by clients. Every relationship needs to answer: *can I safely move from version A to version B?*

Before a shared convention, version numbers were vibes ("2.0-final-FINAL-real"). Downstream consumers had to read changelogs, guess, and discover breakage in production. SemVer encodes **the contract of change into the number itself**.

## Layer 1 — Simple Explanation

A version number is a **medicine label**:

- PATCH: same medicine, corrected dosage — safe to swap
- MINOR: same medicine plus a new vitamin — still safe, new benefits optional
- MAJOR: different medicine entirely — read the label, check interactions

Consumers decide to update by reading only the label — that's the promise.

## Layer 2 — Engineer's View

**SemVer is a public API promise, and it has precise semantics:**

```text
1.4.2  →  1.4.3   OK (patch)
1.4.2  →  1.5.0   OK (minor — new features appeared)
1.4.2  →  2.0.0   BREAKING (something you depend on may have changed)
1.x    →  0.x     ANYTHING can change (0.x = "no stability promise")
```

Note the `0.x` special case — pre-release software makes no compatibility promises, which is why living on `0.x` dependencies is living dangerously.

**The ecosystem machinery that depends on version strings:**

| Mechanism | How it uses versions |
|---|---|
| Maven/Gradle ranges | `dependency: [1.4,2.0)` — resolves compatible versions automatically |
| `^` and `~` (npm) | `^1.4.2` = compatible upgrades within major |
| Lockfiles | freeze the *resolved* graph for reproducible builds |
| Container tags | `myapp:1.4.2` — **tags are mutable unless you enforce immutability** |
| API evolution | REST headers, gRPC/proto changes tied to version policy |

**The three versioning domains — don't confuse them:**

1. **Package/library version** — SemVer fits perfectly (API contract)
2. **Release version** — often calendar or marketing-driven (Ubuntu 24.04, Windows 11); SemVer optional
3. **Build/artifact version** — what DevOps owns; usually SemVer + build metadata:

```text
2.1.0+build.342  or  2.1.0-rc.1+a1b2c3d
```

**The engineering rule you already live by:** *every artifact must be traceable to the exact commit that built it*. Tagging the git commit (`git tag v2.1.0`) and embedding the commit SHA into the artifact creates the bidirectional traceability SDLC demanded: `image:2.1.0` → build 342 → commit `a1b2c3d` → PR → work item. Incident forensics depends on this chain.

**Where SemVer strains:**

- Large systems where "any change might break someone" — every release is de facto MAJOR (leading to CalVer: `2024.05.0`)
- Databases/schema changes — a column rename is breaking for old readers; no number scheme fixes that, migration strategy does
- Human psychology: teams avoid MAJOR bumps by calling breaking changes MINOR, corroding trust — the number is only as honest as the team

## Real-World Example (DevOps flavored)

A classic production incident pattern you've probably seen:

```text
Pipeline: FROM node:latest        # ❌ "latest" = unversioned dependency
          or
Docker tag "latest" overwritten nightly
→ Tuesday morning: build fails / behavior changes. Nothing in your repo changed.
```

Versioning discipline for DevOps:

- Pin base images by digest or exact version: `node:20.11.1@sha256:...`
- **Immutable tags** in your artifact repository — `2.1.0` must never point to different bytes tomorrow
- Git tag → build → artifact version in one pipeline step; never type versions by hand
- Chart/pipeline versioning follows the same rules (Helm `version` vs `appVersion`)

## Common Mistakes

- `latest` as a deployable dependency — a time bomb with no version at all
- Mutating published versions ("fix" a broken `1.4.2` by replacing it) — destroys reproducibility and trust
- Skipping build metadata — two artifacts both called `1.4.2` are indistinguishable in an incident
- Breaking changes snuck in as MINOR
- Versioning only the artifact, never the environment (more in the GitOps phase: environment state must be versioned too)

## Mental Model

> A version number is a **contract printed on the box**. PATCH says "same product, safer." MINOR says "same product, more." MAJOR says "new product — re-read the manual." Ecosystems automate against the contract — which is exactly why lying on the label is catastrophic.

## Remember This

1. MAJOR.MINOR.PATCH = breaking / compatible-feature / fix
2. `0.x` promises nothing; `latest` isn't a version
3. SemVer's real power: dependency resolution and lockfiles automate against it
4. Embed commit SHA + build metadata into every artifact — traceability chain to the work item
5. Artifact tags must be immutable; environments need versioned state too
6. Distinguish package / release / build versioning domains

## One Sentence

Versioning gives every state of software an identity, and SemVer turns that identity into a compatibility contract that dependency tools, pipelines, and consumers can automate against.

## Knowledge Check

1. Why is `0.x` treated differently from `1.x`?
2. Trace the full chain from a running container to the work item that requested its oldest dependency change.
3. Why must a published version tag never be mutated?
4. Your build breaks though "nothing changed" — what versioning sin is the likely cause?

## Further Reading

- [semver.org](https://semver.org) — the spec, 10 minutes
- *Continuous Delivery* — Humble & Farley, chapter 2 (on artifact traceability)

---

**← Previous:** [Code Review](code-review.md)
**Next:** [The Testing Pyramid](testing-pyramid.md) →
**Related:** [CI/CD](../cicd/index.md) · [Shift Left](shift-left-right.md)
