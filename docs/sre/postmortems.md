# Postmortems

## What Is It?

A **blameless postmortem** is the written analysis after an incident (or near-miss): timeline, impact, root *causes* (plural, systemic), action items with owners and dates — producing an org that *learns* from failures instead of hiding them.

```text
The template:
Summary → Impact (users, duration, budget spent) → Timeline (from the scribe)
→ Root causes (systemic: what made this possible, not who tripped)
→ What went well / badly → Action items (owner, date, priority) → Lessons
```

## Why Does It Exist?

Because of the alternative, studied to death in aviation and medicine: **blame culture makes failure invisible**. Punish the person who touches the broken system, and next time nobody reports — same system, same failure, now without a paper trail:

```text
Blameful:  "Alex pushed the bad config" → fix: Alex careful/warned → system unchanged
Blameless: "config lacked validation + deploy lacked canary + rollback took 40 min"
           → fixes: admission policy (PaC), canary (CD), pipeline rollback — SYSTEM changed
```

The aviation lesson: near-perfect safety came when *reporting became safe*. Software adopted it (Google SRE's "blameless" canon) for the same physics.

## Layer 1 — Simple Explanation

The postmortem is the **airline crash investigation**: the NTSB's reports never name or punish pilots — they ask *what in the system made this mistake possible* and issue directives to every airline. The point isn't the one crash; it's making the *next ten thousand flights* safer. The airline that fires pilots teaches its pilots to hide near-misses.

## Layer 2 — Engineer's View

**Root cause discipline — the "5 whys" done honestly:**

```text
Checkout was down 40 min.
why? → payment-svc OOMKilled repeatedly
why? → memory leak in new release
why? → load test used 10x less traffic than prod shapes
why? → perf tests run manually, not in pipeline (Testing page gap)
why? → no gate required them (shift-left as policy, absent)
Action items: perf test in pipeline (owner: platform, 2wk);
              OOM alert on memory trend (SRE, 1wk); canary memory gate (platform, 3wk)
```

Note: no human appears in the causes. "Human error" is rephrased every time as "what made the human's reasonable action dangerous?" — that rephrase *is* the practice.

**The quality bar that separates learning from theater:**

| Symptom of theater | The real thing |
|---|---|
| Root cause: "bad deploy" | Systemic chain (above) |
| Action items: "be careful", "more training" | System changes: gates, automation, alerts |
| No owners/dates | Tracked in the issue tracker like code |
| Only SEV-1s reviewed | Near-misses too (the free lessons — nothing broke *this* time) |
| Meeting to assign weight | Meeting to find the system's weaknesses |

**The follow-through machine (where postmortems die quietly):** action items enter the *normal* backlog with priority reflecting budget burned — and get reviewed. An org with 200 open postmortem items has a museum, not a learning loop. Corollary: if action items never complete, incidents repeat — *that's the measurable*.

**Near-miss culture — the multiplier:** a flag-off-in-30-seconds incident is a *free preview* of the SEV-1 it would have been. Postmortem-worthy, blamelessly, because the system gap was real even though the luck was good.

## Real-World Example (DevOps flavored)

The quarterly review that proves the loop works:

```text
Q3: 6 postmortems (2 SEV-1), 19 action items, 17 completed
Recurring theme detected: 4/6 incidents involved "config change without validation"
Meta-action: config-schema admission policy (Kyverno — Policy page) deployed org-wide
Q4: 2 postmortems, none config-related — the class of incident was eliminated,
     not just the instances. That's an org compounding its learning.
```

## Common Mistakes

- Blame leaking in ("the on-call should have...") — reports go silent, learning stops
- "Root cause: bad deploy" — the analysis that isn't one
- Action items as unowned aspirations
- Reviewing only the big ones — near-misses are the free tuition
- Postmortem as punishment ritual — the meeting people dread instead of request
- Writing them a week later — timelines decay; write within 48h (scribe's notes are fresh)

## Mental Model

> The postmortem is the **NTSB investigation**: never about punishing the pilot, always about the directive that changes every future flight. Near-misses are the **free lessons** — the crash you got to study without paying for it. An org that punishes reporters has built a museum of recurring incidents with the lights off.

## Remember This

1. Blameless is a *systems* stance: rephrase every human error as the system gap that allowed it
2. Root causes are chains (5 whys), never single events or persons
3. Action items: systemic (gates/automation/alerts), owned, dated, tracked in the backlog
4. Near-misses get postmortems — free lessons
5. Completion rate of items = the loop's health metric; open items = museum
6. Write within 48h from the scribe's live timeline; review for themes quarterly

## One Sentence

A blameless postmortem converts incidents and near-misses into systemic learning — timeline, causal chain, and owned corrective actions — on the understanding that organizations that punish reporters simply stop hearing about failure.

## Knowledge Check

1. Rewrite "the engineer ignored the alert" blamelessly, twice (two system gaps).
2. Why are near-misses the highest-ROI postmortems?
3. Your team's last 10 action items: 3 completed. Diagnose and fix the loop.
4. What quarterly meta-analysis turns individual postmortems into org-level learning?

## Further Reading

- Google SRE book ch. 15 (postmortem culture) — the source
- Etsy's Blameless Postmortems (Morgenthaler) — the industry classic essay
- Next: [Capacity Planning](capacity-planning.md)

---

**← Previous:** [Incident Management](incident-management.md)
**Next:** [Capacity Planning & Scalability](capacity-planning.md) →
**Related:** [Incident Management](incident-management.md) · [Vulnerability Management](../security/vulnerability-management.md)
