# AI System Architecture — The Full Stack

## What is it?

Every page of this track, assembled into one production diagram. If you can explain every layer of this stack and who owns it, you can design AI systems.

```text
User
 ↓
Application (UI, product logic)
 ↓
AI Gateway          — auth, routing, model selection, rate limits, cost caps, caching
 ↓
Context Engine      — prompt assembly, RAG retrieval, memory, compression (Context page)
 ↓
Model               — the frozen artifact (local engine or provider API)
 ↓
Tools / Actions     — validated tool registry; MCP servers; approval gates
 ↓
Orchestration       — agent loops / graphs; supervisor patterns; HITL gates
 ↓
Guardrails          — injection filters, PII, moderation, egress control
 ↓
Observability       — traces of every step; token/cost/quality telemetry
 ↓
Evaluations         — golden sets in CI; online quality monitoring
```

## The AI Gateway (the layer your DevOps instincts built)

The pattern crystallizing in every serious org — an **AI gateway** as the control plane:

| Responsibility | Analog from the main track |
|---|---|
| AuthN/Z per consumer | API gateway (Networking phase) |
| Model routing (task → cheap/frontier; fallback) | tiered routing + circuit breakers |
| Cost: budgets per team, per feature | showback, quotas (FinOps) |
| Caching (identical prompts) | CDN semantics for tokens |
| Provider abstraction | avoids lock-in; base-URL swap (Running-models page) |
| Audit logging | every call: prompt version, model version, tokens, result |

One gateway = one place to enforce cost, security, and observability for all AI features — the platform-as-product move applied to AI.

## Design decisions at the seams

| Seam | The recurring question |
|---|---|
| App ↔ gateway | who pays, who's rate-limited, what's cached |
| Context ↔ model | local vs hosted (Open/Closed page's table, per feature) |
| Model ↔ tools | tool scopes per feature — not per user! |
| Orchestration ↔ humans | approval gates by action class (Agentic page) |
| Everything ↔ observability | if it's not traced, it didn't happen |

## The knowledge map (the whole track, one diagram)

```text
Rules → ML → Neural Nets → Deep Learning → NLP → Embeddings
  → Attention → Transformers → Scaling → LLM → Alignment
    → Inference (tokens, KV cache, quantization, hardware)
      → Prompts → Context → Structured Outputs → Tools → RAG
        → Evals → Guardrails → Agents (loops/graphs/harnesses/memory/multi-agent)
          → Architecture (this page) → Observability → Security → Cost
            → Reliability → LLMOps → Capstone
```

Each arrow is a page you've read; each was a limitation forcing an invention.

## The DevOps connection (both tracks meet)

```text
Software Engineering + DevOps + Cloud + K8s + Security + SRE + Platform Engineering
        = the harness layers (gateway, serving, observability, guardrails, delivery)
AI Engineering = the model/context/agent layers
        → combined: modern AI-powered engineering platforms
```

Your six years are the bottom half of the stack. This track was the top half. The capstone (end of this section) builds the whole thing.

## Remember This

1. The stack: app → gateway → context engine → model → tools → orchestration → guardrails → observability → evals
2. The AI gateway is the control plane: routing, cost, caching, audit — one enforcement point
3. Decisions live at the seams — especially model placement (local vs hosted, per feature)
4. The knowledge map's arrows are limitations-forcing-inventions — you can now draw it from memory
5. Both tracks combine into one platform — you own both halves

## One Sentence

A production AI system is a layered architecture — gateway, context engine, model, tools, orchestration, guardrails, observability, evaluation — where every layer is a concept from this track and the lower half is the DevOps platform you already know how to build and operate.

## Knowledge Check

1. Draw the stack from memory; name the owner of each layer in a platform-team org.
2. Which gateway responsibility would you build first, and which incident does it prevent?
3. Trace one user question through all nine layers, naming what each adds.

---

**← Previous:** [Coding Agents](coding-agents.md)
**Next:** [AI Observability](ai-observability.md) →
