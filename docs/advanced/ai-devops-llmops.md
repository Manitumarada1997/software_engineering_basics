# AI for DevOps, AI Engineering & LLMOps

## What Is It?

Three overlapping territories at the course's frontier:

- **AI for DevOps**: applying AI to *your* work — incident summarization, log triage, test generation, pipeline repair
- **AI engineering**: building features *with* LLMs — a new programming model: natural language + context → probabilistic output
- **LLMOps/MLOps**: operating AI systems — the delivery, evaluation, and monitoring of models and their pipelines

```text
Your 6 years of DevOps + this course = the exact skill set LLMOps lacks most:
LLM systems are production systems — delivery, observability, cost, security —
with two new primitives: evaluation and prompts-as-code.
```

## Why Does It Exist?

Two forces converging on people with your background:

**1. LLMs broke software's determinism contract.** Traditional engineering (this entire course) assumes: same input → same output → tests assert, pipelines gate. LLM features are *probabilistic*: the same prompt yields varying quality. That breaks testing (Testing page), CI gates (CI page), and monitoring (what's "correct"?) — requiring new machinery: **evals** (graded test-suites for outputs), **guardrails**, and quality-observability (sampling human review, model-based scoring). LLMOps is CI/CD + SRE rebuilt for non-determinism.

**2. AI is now load-bearing in engineering orgs** — code assistants, incident copilots, customer-facing bots — which makes it *infrastructure you operate*: versioned (models move under you — "model drift" is your dependency drift page's cousin), metered (tokens = the FinOps page's new line item), secured (prompt injection = the injection class, new surface).

## Layer 1 — Simple Explanation

- **AI for DevOps**: the **apprentice who read every runbook** — drafts your incident summary, triages the log flood, writes the test skeleton; you review (shift-left with a tireless junior)
- **LLMOps**: operating the **factory where outputs are opinions** — quality control (evals) replaces unit tests, sampling inspection replaces assertion gates, and raw materials (models) change supplier without asking

## Layer 2 — Engineer's View)

**The new primitives (mapped to your existing ones):**

| New | Maps to | The difference |
|---|---|---|
| **Evals** (graded output suites) | test suites | graded (LLM-judge/human), probabilistic pass, thresholds not binaries |
| **Prompts + context** | code | versioned, reviewed, A/B-able — but English-shaped; injection-reachable |
| **Model versions** | dependencies | upstream changes behavior silently; pin + regression-eval |
| **Guardrails** | policy-as-code | output/injection filters as gates |
| **Tokens** | compute quota | the metered unit — cost, rate limits, budget alerts |
| **Grounding (RAG)** | caching, joins | retrieve facts into context — the hallucination defense |

**The LLMOps pipeline (your pipeline, extended):**

```text
prompt/context change (PR) → eval suite runs (graded thresholds) → canary with
  quality-metrics sampling → monitor: quality drift, cost/token burn, latency,
  injection alerts → rollback = model/prompt version pin
New observability: trace = prompt+context+output (OTel spans exist for this),
  quality dashboards with human-sampling loops, cost per feature
```

**Security — the part orgs skip (your Security phase's new chapter):**

- **Prompt injection** = the new injection class: untrusted text (user input, *retrieved documents*) steering the model — treat retrieved content as untrusted input, always (Zero Trust, applied to context)
- Data exfiltration via model calls (the "summarize this and call that URL" attack) — egress control applies
- Secrets in prompts; PII in context sent to third-party models (data-residency law applies — the Regions page's sovereignty, again)

**MCP (Model Context Protocol)** — one paragraph, since your roadmap asked: the emerging standard for connecting models to tools/data sources (context, APIs, actions) — poised to do for AI-tool integration what OCI did for containers and OTel for telemetry: *standardize the interface, compete on implementation*. Worth tracking the same way you tracked Gateway API.

## Real-World Example (DevOps flavored)

ShopEasy's AI, run the way this course would insist:

```text
Support copilot:  RAG over runbooks; evals = 200 graded Q/A (threshold: 90% helpful);
                  prompt+context versioned in Git; canary 5% with quality sampling;
                  injection guardrails (retrieved docs = untrusted); cost cap/alerts
Incident copilot: draft summaries from timelines + traces — human edits measured
                  (edit-distance falling = trust growing — the DX metric pattern)
LLMOps stack:     OTel GenAI spans → quality/cost/latency dashboards per feature
Result discipline: model upgraded → regression evals caught 3 quality drops in a quarter
```

## Common Mistakes

- Shipping LLM features with no evals — "it seems fine" as the quality gate
- Prompts as unversioned folklore (config drift, English edition)
- Treating retrieved/context content as trusted — the injection door left open
- No cost telemetry — token burn discovered by finance (the FinOps failure, replayed)
- Monitoring infra metrics but not *quality* drift — degradation invisible until users complain
- Believing AI for DevOps replaces understanding — the apprentice drafts; the journeyman (you, after this course) reviews

## Mental Model

> LLM systems are **factories where products are opinions**: quality control by sampling and grading (evals) instead of assertion, raw materials that change supplier mid-run (model versions), and a new shop-floor hazard (injection through the context door). Everything else — pipelines, observability, FinOps, security layers — is the factory you already know how to run.

## Remember This

1. Three territories: AI-assisted DevOps, AI engineering, LLMOps — your skills transfer directly
2. Core break: non-determinism → evals, thresholds, sampling replace assertions
3. Prompts/context = code: versioned, reviewed, injected-against
4. Model versions are dependencies: pin + regression-eval on upgrades
5. Prompt injection: retrieved content is untrusted input — Zero Trust for context
6. Tokens are the meter: cost telemetry and quality monitoring from day one

## One Sentence

AI engineering inherits your entire delivery discipline — pipelines, observability, security, cost — and adds the machinery non-determinism demands: graded evals instead of assertions, versioned prompts instead of folklore, and injection-aware trust boundaries around everything the model touches.

## Knowledge Check

1. Map each LLMOps primitive to its closest precedent in this course (evals→?, model-drift→?, tokens→?).
2. Design the eval + canary + monitoring plan for a customer-facing support bot.
3. Why is retrieved RAG content an injection vector? Which Zero Trust rule applies?
4. Your model vendor upgrades silently tonight. What catches it Monday morning?

## Further Reading

- [OTel GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) · Anthropic/OpenAI ops guides
- [modelcontextprotocol.io](https://modelcontextprotocol.io/) — MCP, the emerging standard
- Final page: [The Capstone](../projects/capstone.md)

---

**← Previous:** [FinOps](finops.md)
**Next:** [The Capstone](../projects/capstone.md) →
**Related:** [Policy as Code](../security/policy-as-code.md) · [Observability](../sre/observability.md)
