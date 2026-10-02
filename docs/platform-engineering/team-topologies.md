# Team Topologies

## What Is It?

Skelton & Pais's (2019) vocabulary for **organizational design as architecture**: four team types and three interaction modes — a language for the org shapes that make software structures work (or doom them).

```text
Team types:
  stream-aligned    — owns a slice of product end-to-end (the value stream)
  platform          — builds the IDP that stream teams consume (previous page)
  enabling          — coaches a stream team past a capability gap, then leaves
  complicated-subsystem — owns one deep specialist component (e.g., the search engine)

Interaction modes:
  collaboration   — two teams working closely, temporarily (learning, new territory)
  X-as-a-Service  — consuming a product with a contract (the platform's mode)
  facilitating    — helping a team help itself (the enabling team's mode)
```

## Why Does It Exist?

Because **Conway's law runs both ways** (Monolith page): org structure *determines* system structure — so designing the org IS designing the architecture. Team Topologies is the reverse-Conway maneuver: *choose the architecture you want, then arrange teams to force it into existence.*

The law's corollary the book formalizes — **cognitive load as the team-sizing constraint**: a team can hold roughly one "team-sized" domain. Teams owning five domains produce five mediocre domains (the Why-Platform page's explosion, org-theoretically stated).

## Layer 1 — Simple Explanation

The four types as a **restaurant group**: the *bistro teams* (stream-aligned) each run a complete restaurant for their neighborhood; the *central commissary* (platform) supplies standardized components-as-a-service; the *consulting chef* (enabling) teaches your kitchen sous-vide, then leaves; and the *pastry lab* (complicated-subsystem) exists because laminated dough is a specialty no bistro should each master.

The modes are the *contract*: bistros don't redesign the commissary (X-as-a-Service); a bistro and the commissary might co-develop a new line (collaboration — temporary!); the consulting chef teaches, never takes over the kitchen (facilitating).

## Layer 2 — Engineer's View

**The stream-aligned team, defined (the book's heart):**

```text
One value stream ("checkout", "search") · owns build AND run (DevOps's "you build
it, you run it" — kept viable by the platform absorbing everything undifferentiated)
· sized by domain, not headcount math · long-lived (owns the stream, not the project)
```

**Platform team constraints (the discipline that keeps it a product):**

- Serves via **X-as-a-Service**: self-service, contractual, versioned — not via ticket queue (the queue form is the failure mode: platform-as-bottleneck)
- Explicit responsibility *scope* — the IDP's capabilities list, no scope creep into product decisions
- Measured on adoption + stream-team throughput (the product metrics — next pages)

**Enabling teams — the mode everyone under-uses:** a security or K8s expert embedded *temporarily* to level up a stream team (shift-left as staffing), whose success metric is *their own departure*. Anti-pattern: the enabling team that stays, doing — now a dependency wearing a coaching hat.

**The interaction-mode hygiene rules:**

| Rule | Why |
|---|---|
| Collaboration is temporary | two teams fused permanently = a bigger, slower team |
| X-as-a-Service needs a real contract | the API-design page, internally |
| Minimize team-API surface | every inter-team dependency is coupling (the Monolith page, human edition) |
| Modes are chosen per-need, reviewed | org design is continuous, like architecture |

**Team API (the book's most portable concept):** each team publishes its interface — what it owns, how to request, SLAs, roadmaps — the API-design discipline applied to teams. "Who do I ask about X?" becomes a lookup, not an archaeology.

**Conway diagnostics (the review questions):**

- Four teams touching every service? → boundaries drawn by layer, not stream
- Every feature needs three teams' sign-off? → missing stream-aligned ownership
- Platform queue is the bottleneck? → X-as-a-Service regressed into ticketing
- Your "enabling" team has existed for 3 years? → it's a dependency now

## Real-World Example (DevOps flavored)

ShopEasy's org (the company you've followed, finally drawn as teams):

```text
Stream-aligned (×9): checkout, search, payments, fulfillment, growth... 
Platform (×1, 6 eng): the IDP — delivery, runtime, observability golden paths
Enabling (×2, rotating): SRE-practices, security-champions — 3-month engagements,
                        success = stream team doesn't need them anymore
Complicated-subsystem (×1): search-index engineering (the deep thing)
Team APIs published in the portal (Backstage — next pages); collaboration mode
    used twice this year (new payment rail; mesh adoption) — both time-boxed
```

The change that mattered: the "shared infra" team of 12 (everyone's dependency, everyone's bottleneck) became the platform product — and the org chart started agreeing with the architecture.

## Common Mistakes

- Layer teams ("backend team", "QA team") across streams — every feature crosses N teams
- Platform as shared-services queue — the bottleneck institutionalized
- Permanent collaboration mode — two teams that are secretly one slow team
- Enabling teams that never leave — dependency in coach's clothing
- Copying the topology without the load-sizing: teams of 15 owning 6 domains

## Mental Model

> Team Topologies is **org design as city planning**: bistro teams own streets end-to-end; the commissary serves components-as-a-service with contracts; consulting chefs teach and leave; the pastry lab guards one deep craft. Conway's law is gravity — this is the counterweight that steers what gravity builds.

## Remember This

1. Four types: stream-aligned, platform, enabling, complicated-subsystem
2. Three modes: collaboration (temporary!), X-as-a-Service (contractual), facilitating
3. Cognitive load sizes teams: one team, one team-sized domain
4. Reverse-Conway: choose the architecture, arrange the org to produce it
5. Team APIs = interface discipline applied to teams
6. Diagnostics: layer-teams, platform queues, permanent collaborations, resident enablers

## One Sentence

Team Topologies provides the organizational vocabulary — four team types, three interaction modes, sized by cognitive load — that turns Conway's law from gravity you suffer into gravity you steer, making the platform's product model structurally possible.

## Knowledge Check

1. Diagnose your org: which type is your team, really? Which mode is your platform consumed through?
2. Why is permanent collaboration a smell? What was the actual missing structure?
3. Design the enabling-team engagement for "stream teams own their SLOs" — entry, exit criteria.
4. Where does your org violate one-team-one-domain, and what does it cost per feature?

## Further Reading

- *Team Topologies* — Skelton & Pais (the book; short, dense, essential)
- [teamtopologies.com](https://teamtopologies.com/) — concepts + case studies
- Next: [Developer Experience & Golden Paths](dx-golden-paths.md)

---

**← Previous:** [Why Platform Engineering?](why-platform-engineering.md)
**Next:** [Developer Experience & Golden Paths](dx-golden-paths.md) →
**Related:** [Monolith → Microservices](../architecture/monolith-microservices.md)
