# AI + DevOps — AI in Your Day Job

## What is it?

The closing loop: AI applied *to* the practices this site taught — incident response, pipelines, troubleshooting, infrastructure — with the risks of AI touching production named honestly. Everything here is an agent/harness (Section 6) applied to tools you already run.

## The applications (each = this track, applied)

| Domain | What AI genuinely helps with | Why it works |
|---|---|---|
| **Incident response** | draft summaries from timelines, correlate traces/logs, suggest first steps, write the postmortem skeleton | the Observability page's telemetry, fed to a reader |
| **Log/triage analysis** | cluster error patterns, surface the anomaly in 10k lines, translate to plain language | pattern-finding at volumes humans skim |
| **Pipelines** | generate/repair YAML, explain failures, suggest cache fixes | templates + docs are its home turf |
| **K8s troubleshooting** | "describe pod events → ranked hypotheses → suggested kubectl" — the Troubleshooting page's method, accelerated | the triage tree is *text* — ideal LLM food |
| **IaC review** | security smells in Terraform plans, drift explanations | plan diffs + policy patterns |
| **Security analysis** | vuln summaries, threat-model drafting (STRIDE pass as an assistant) | checklists + reading comprehension |

The honest pattern in every row: **AI as the reader/drafter, engineer as the decider** — exactly the dispatcher/executor split (Tool-calling page). It accelerates the *language-heavy* 40% of ops work; the *consequence-carrying* 60% stays yours.

## AI operating infrastructure (the autonomy question)

The escalation (each step = more agency, more guards):

```text
read-only analyst   — queries, summarizes, recommends        (safe; ship today)
gated actor         — proposes actions; human approves       (the HITL default)
bounded autonomy    — auto-remediates known-safe classes     (restart, scale-out)
                     with full audit + reversal
full autonomy       — reflexes on production                 (don't — yet)
```

The K8s-phase lesson holds: start where the blast radius is zero (reads), earn autonomy per action-class with verification. An ops agent's tools should look like: `get_events`, `get_logs`, `describe`, `rollout_undo` (gated), `scale` (gated) — *scoped verbs, never a shell*.

## The risks (production actions by a non-deterministic dispatcher)

- **Confidently wrong diagnosis → wrong (approved) action**: the approver must verify against telemetry, not prose fluency — dashboards in the approval loop, not just the agent's summary
- **Injection through logs/events**: your cluster's event text is untrusted input (a pod name can carry instructions — the Guardrails law reaches into `kubectl` output)
- **Credential scope creep**: the agent's service account is the crown — least privilege, audited, exactly the IAM page
- **Atrophy**: teams that stop reading the dashboards because the agent summarizes — keep humans in the telemetry, not just above it
- **Cost of the assistant**: an ops agent in a loop during an incident = tokens × stress; budget caps (Loops page)

## Where you go from here (both tracks complete)

You now hold: the DevOps platform disciplines (main track) *and* the AI stack (this track). The capstone — one page away — asks you to build the intersection: an AI-powered assistant over a real platform, with every layer this site taught.

## Remember This

1. AI accelerates the language-heavy slice of ops: summaries, triage, drafts — engineer decides
2. Dispatcher/executor: the agent reads and recommends; scoped verbs, never a shell
3. Autonomy ladder: read-only → gated → bounded classes → (not yet) full
4. Injection reaches through logs and events — cluster output is untrusted input
5. The agent's service account is the crown jewel — IAM discipline, verbatim

## One Sentence

AI in DevOps means non-deterministic readers and drafters attached to your telemetry with scoped, gated, audited tools — accelerating diagnosis and drafting while engineers keep every consequence-carrying decision, earning autonomy one verified action-class at a time.

## Knowledge Check

1. Design the tool registry (names + scopes + gates) for a read-mostly ops agent on your cluster.
2. Which of your daily tasks is language-heavy and verification-rich — the sweet spot?
3. Construct the injection-through-pod-name attack and the control that neutralizes it.

---

**← Previous:** [AI Workflow](ai-workflow.md)
**Next:** [Capstone — The ShopEasy Assistant](capstone-shopeasy-assistant.md) →
