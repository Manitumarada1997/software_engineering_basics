# Tool Calling

## What is it?

**Tool calling** (function calling): the model, instead of answering directly, emits a *structured request* — "call `get_pod_status(name="payments-7d9f")`" — which **your code** executes; the result goes back into context; the model continues. The model never touches your systems; it fills out a form. You run the form.

## The problem it solves (the Limits page's "no actions", concretely)

An LLM alone cannot check a pod, query a DB, or fetch a ticket — it predicts tokens. Products needed: "what's wrong with payments?" answered from *live cluster state*, not frozen training data. Tool calling turns prediction into **delegation**: the model decides *what* would help; your infrastructure performs it.

## The loop (memorize this shape — it powers everything ahead)

```text
User: "why is payments slow?"
  1. Model emits a TOOL CALL (structured output):
     get_metrics{service:"payments", metric:"p95_latency", window:"30m"}   ← tokens only
  2. Your code executes the real call (kubectl/prometheus API)
  3. Result → back into the context window
  4. Model, now grounded, emits the next step or the final answer
```

Steps 2–3 are *your engineering*: the model's tool vocabulary is a schema you declare (names, typed parameters, descriptions); execution, auth, and error handling are your code. **The model is the dispatcher; you are the executor** — and everything about security follows from that split.

## Designing tools well (the craft)

| Rule | Why |
|---|---|
| Small, single-purpose functions | the model reasons better over focused verbs |
| Descriptions are prompts | the model *reads* them to decide — bad docs = wrong tool |
| Typed parameters + enums | structured outputs' rules, verbatim |
| Return distilled results | tool output enters the paid-for context window — compact JSON, not raw dumps |
| Idempotent where possible | retries happen; make them safe (the distributed-systems law) |
| Error messages written for the model | "invalid: cluster name must match ..." — the model self-corrects on retry |

The last two are underrated: a tool's error text is a *prompt*, and a good error message lets the loop recover without human intervention.

## The security implications (the page's most serious section)

Tool calling is where tokens gain the ability to affect reality — and where all your DevSecOps discipline becomes mandatory:

- **Least privilege per tool**: the metrics tool gets metrics read; it does not get cluster-admin. The model's permission is the tool's permission — scope ruthlessly
- **Allowlist, never freeform**: models call *declared* tools only; no "execute this command" tools
- **Validate arguments at execution**: the model constructs the arguments — treat them like user input (they may be attacker-shaped: injection, next section)
- **Human approval for destructive actions**: delete/scale-down/spend — a gate, not a hope
- **Audit every call**: who, what tool, what args, what result — the forensic trail
- **Rate/cost limits**: tool loops can spiral; cap iterations and spend

This is the Security phase's zero-trust doctrine with the model as an untrusted-but-useful caller.

## The DevOps mapping

| Tool calling | Your world |
|---|---|
| Tool schema | API contract (OpenAPI) — the model is a client |
| Execution split | a dispatcher and an executor — the model never runs code |
| Error-as-prompt | good error messages make systems self-healing |
| Allowlists + audit | RBAC + audit logs — unchanged by the model's presence |

## What came next

One tool call answering one question is useful. But "investigate why payments is degraded" needs *many* calls, chosen adaptively — that's a **loop with a goal**, and that is an agent: the next section's subject.

## Remember This

1. The model emits structured requests; your code executes — dispatcher/executor split, always
2. Tool vocabulary = schemas you declare; descriptions are prompts
3. Results enter context: return distilled JSON, write model-readable errors, idempotent
4. Security: least-privilege per tool, allowlists, validate args, human gates on destructive ops, audit all
5. Tool calling is the seed of agents — loops come next

## One Sentence

Tool calling lets a model affect the world by delegation — emitting typed tool requests that your validated, least-privileged, audited code executes — converting prediction into grounded action while keeping execution authority in engineering hands.

## Knowledge Check

1. Trace the four steps for "restart the payments pod" — where exactly does human approval belong?
2. Why is "a shell tool" the antithesis of good tool design?
3. Rewrite a cryptic tool error into one a model can self-correct from.

---

**← Previous:** [Structured Outputs](structured-outputs.md)
**Next:** [RAG](rag.md) →
