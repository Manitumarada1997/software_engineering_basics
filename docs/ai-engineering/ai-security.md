# AI Security — The Full Threat Model

## What is it?

The Guardrails page built the injection defense; this page assembles the complete threat model for agentic systems and maps every control to your DevSecOps discipline. One law governs everything:

> **Anything the model reads is an attack channel. Anything the model can do is an attack objective.**

## The threat catalog (each with its control)

| Threat | Mechanism | Control |
|---|---|---|
| **Prompt injection** | instructions smuggled in user text | input filters, labeled channels (Guardrails page) |
| **Indirect injection** | malicious instructions in retrieved docs/tickets/emails/code | untrusted-content law; retrieval source allowlists |
| **Jailbreaks** | crafted prompts to bypass safety | defense-in-depth; accept probabilistic loss; outer filters |
| **Data leakage** | secrets/PII leaving via model output or tools | PII redaction both directions; egress control |
| **Tool abuse** | model tricked into harmful tool use | least privilege per tool; allowlists; argument validation |
| **Excessive agency** | the agent *can* delete/spend/notify | scope tools to intent; approval gates; dry-runs |
| **Agent hijacking** | persistent steering via poisoned memory | Memory page: curated writes, tenancy, revocation |
| **RAG poisoning** | planted documents steering retrieval | ingestion provenance, doc signing, source gating |
| **Model supply chain** | poisoned/stolen/unproven model weights | checksums, reputable registries, weight provenance |
| **MCP/server supply chain** | malicious capability servers | vet, pin, gateway, audit (MCP page) |
| **Confused deputy** | agent's cloud creds abused via SSRF-style paths | the IAM page's chain — unchanged, now model-reachable |

## Securing agentic systems (the principles, translated)

Every principle is your existing security phase, verbatim, one row each:

| Principle | DevSecOps original | AI application |
|---|---|---|
| **Least privilege** | IAM scopes | per-tool, per-agent credentials — the agent is a service account |
| **Sandboxing** | containers/isolation | tool execution in sandboxes; network-limited |
| **Allowlists** | policy-as-code | declared tools only; declared retrieval sources only |
| **Permission boundaries** | RBAC | who may *deploy* agents/tools — Tier-0 thinking |
| **Human approval** | CD gates | destructive/spend actions gated (HITL) |
| **Input/output validation** | never trust input | both directions around the model — schemas |
| **Isolation** | tiers/segments | per-tenant agents, memory, retrieval |
| **Monitoring & audit** | SIEM/CloudTrail | every model call, tool call, decision logged |
| **Zero trust** | NIST 800-207 | "inside the model" ≠ trusted; every channel verified |

## The design exercise that summarizes the track (walk it)

Support agent that reads tickets (external text), retrieves runbooks (internal docs), queries the cluster (tools), and emails users (action). Threat walk:

1. Ticket text = untrusted → labeled channel; never instructions
2. Runbook wiki = semi-trusted → source allowlist; provenance (RAG poisoning)
3. Cluster tools = read-only service account (least privilege); args validated
4. Email tool = human-approval-gated for external recipients (excessive agency)
5. All steps traced; anomaly alerts on weird tool sequences (hijacking)
6. Memory extracted only from curated sources, tenant-scoped (poisoning)

Six sentences, six controls, all from pages you've read. That's AI security engineering.

## The DevSecOps mapping (the punchline)

You do not need a new security discipline. You need your existing one, applied to two new surfaces (context as input; tools as actions) and one new dependency class (models/servers as supply chain). The org that secures K8s well secures agentic AI well — same muscles, new anatomy.

## Remember This

1. The governing law: everything read is a channel; everything doable is an objective
2. The catalog is ~10 threats; every one maps to an existing DevSecOps control
3. Agent = service account: least privilege, sandboxed, audited, gated
4. Supply chain extends to weights and MCP servers — provenance and pinning
5. Security review of an AI feature = the six-sentence walk above

## One Sentence

AI security is your DevSecOps discipline applied to new surfaces — context as an attack channel, tools as attack objectives, and models as supply chain — governed by least privilege, validation, sandboxing, approval gates, and total audit, exactly as before.

## Knowledge Check

1. Run the six-sentence threat walk for an agent that reads PRs and modifies CI pipelines.
2. Why is "the agent's credentials" the highest-leverage control in the whole model?
3. Which threat only exists because of persistence (memory)? Name its controls.

---

**← Previous:** [AI Observability](ai-observability.md)
**Next:** [AI Cost](ai-cost.md) →
