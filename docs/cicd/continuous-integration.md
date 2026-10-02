# Continuous Integration (CI)

## What Is It?

Continuous Integration is the practice of **merging every developer's work into the shared mainline at least daily**, with each merge **verified by an automated build and test run**.

Two halves, both required:

```text
CI = frequent integration (people/process)
   + automated verification (build + tests, machines)
```

Frequent merging without tests = chaos on main. Tests without frequent merging = a test suite guarding a museum. Named and popularized from XP (Kent Beck, late 1990s) — recall "if integration is good, integrate constantly."

## Why Does It Exist?

Because integration is where software projects die. Pre-CI, teams worked on private branches for weeks, then suffered "integration hell":

```text
Week 1–6:  everyone develops happily in isolation
Week 7:    merge week. 3,000 conflicts. The combined build has never existed.
Week 8–10: stabilization. New features frozen. Everyone miserable.
```

CI's inversion: **make integration a non-event.** Integrate daily, let the machine catch incompatibilities within hours of creating them, while the change is fresh and small (the cost-of-change curve from the Waterfall page, applied to integration).

## Layer 1 — Simple Explanation

Old way: five families renovating five rooms of the same house for a month, then meeting in the hallway to discover nobody can open the front door. CI way: every evening, all work meets on-site; small collisions get fixed the same evening while everyone remembers what they did.

## Layer 2 — Engineer's View

**The CI contract (what "doing CI" actually means):**

1. Mainline exists and is **always buildable** — a red main is a stop-the-line event
2. Every push triggers: compile → unit tests → static analysis → artifact build
3. Failures are fixed **within minutes/hours**, not "eventually"
4. The artifact produced is **versioned and stored** — CI always produces evidence

**The pipeline anatomy you already operate — mapped to purpose:**

```mermaid
flowchart LR
    PR[PR opened] --> V[Validation build<br/>+ tests + SonarQube gate]
    V -->|green| M[Merge to main]
    M --> C[CI pipeline: build + test]
    C --> A[Versioned artifact → repository]
    A --> D[Deploy staging (CD begins)]
    V -->|red| X[Fix within the hour]
```

**The metrics that define CI health (DORA):**

| Metric | Meaning | Elite target |
|---|---|---|
| Lead time | commit → production | < 1 day |
| Deployment frequency | how often you release | on demand, multiple/day |
| Change failure rate | % of deployments causing failure | 0–15% |
| MTTR | time to restore | < 1 hour |

CI speed specifically: **the faster the feedback, the smaller the effective batch**, and batch size is the master variable of delivery (Little's Law again). A 10-minute CI loop changes developer *behavior*; a 90-minute one forces batching, and batching reintroduces integration risk.

**Branch/PR validation nuance:** true CI integrates *on main*. PR validation pipelines are the gate that makes merging safe (merge queues serialize concurrent PRs, testing each against the result of the last — the Google Zuul pattern).

**Build hygiene — the forgotten half of CI:**

- Flaky tests = the boy who cried wolf; quarantine immediately, fix or delete
- Deterministic builds (previous page) — or "CI green" means nothing
- Secrets via OIDC/federation, never stored on agents (Security phase)
- Agent hygiene: ephemeral, immutable, auto-scaled (this is where DevOps *owns* CI as a product)

## Real-World Example (DevOps flavored)

Azure DevOps, concretely:

```yaml
# ci.yml — the whole CI contract in ~15 lines
trigger: [ main ]
pr: [ main ]
pool: { vmImage: ubuntu-latest }
steps:
  - task: Cache@2          # deps
  - script: mvn -B verify  # build + unit + integration tests
  - task: SonarQubeAnalyze@1  # quality gate
  - script: mvn -B package deploy -Dversion=$(Build.BuildId)
```

What you add as the platform engineer: caching, parallelized test shards (drop 20 min → 4), merge queue policy, agent autoscaling, required status checks. Every minute you shave multiplies by every developer × every push — CI speed is a *team-wide productivity lever*, not an infrastructure vanity metric.

## Common Mistakes

- CI = "we have Jenkins" — the tool without the practice (infrequent merges, red main for days) is theater
- Skipping the artifact: build + test + *discard* — then CD has nothing versioned to deploy
- Tolerating flaky tests until developers auto-retry red builds (trust destroyed)
- Nightly-only builds — that's scheduled integration, not continuous
- CI on main but developers living on week-old feature branches (the actual integration still isn't continuous)
- Tests that require manual setup ("the DB must be seeded by hand") — not CI, a demo

## Mental Model

> CI is **brushing your teeth daily** vs. one traumatic root-canal quarter. Small, boring, frequent maintenance instead of episodic surgery. The pipeline is the toothbrush; a red main is "you have something stuck in your teeth *right now*."

## Remember This

1. CI = frequent merges + automated verification — both halves required
2. Kills integration hell by making conflicts small, fresh, and machine-detected
3. Main must always build; broken main stops the line
4. CI always produces a versioned artifact — it's CD's input, not just a green check
5. Feedback speed shapes batch size shapes delivery performance (DORA)
6. Flaky tests destroy the trust the whole system runs on

## One Sentence

Continuous Integration makes merging code a routine, machine-verified event by integrating at least daily and proving every merge with an automated build that produces a versioned artifact.

## Knowledge Check

1. Why is "frequent merge + no tests" and "tests + monthly merge" both broken CI?
2. Explain the causal chain: CI duration → batch size → integration risk.
3. What does CI produce that CD consumes, and why must it be versioned?
4. What are the four DORA metrics, and which two does CI most directly drive?

## Further Reading

- Martin Fowler, *Continuous Integration* (martinfowler.com, the classic 2006 article + 2024 update)
- *Accelerate* — Forsgren, Humble, Kim
- DORA reports —dora.dev

---

**← Previous:** [Build Automation](build-automation.md)
**Next:** [Artifact Repositories](artifact-repositories.md) →
**Related:** [Trunk-Based Development](../development-practices/trunk-based.md) · [Continuous Delivery](continuous-delivery.md)
