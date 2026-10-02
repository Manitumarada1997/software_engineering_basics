# Structured Outputs

## What is it?

Getting an LLM to return **machine-parseable output** — JSON matching a schema — reliably, instead of free text that your code must hope and parse. The core technology for making LLMs a *component* in software rather than a chat toy.

## The problem

You want `{"severity": "high", "service": "payments"}`; the model, left alone, produces prose, markdown fences, occasional apologies, and — inevitably — the trailing comma of doom at 3 AM. Free-text sampling (Generation page) has no format guarantee *by construction*. Your code needs a contract; raw generation doesn't offer one.

## The solutions, in layers of reliability

**1. Prompt-for-format (weakest):** "reply ONLY with JSON matching this schema." Helps, breaks under pressure — long answers drift, creative modes leak markdown. Treat as prototyping only.

**2. Schema-constrained generation (strong):** modern APIs accept a **JSON schema** and the *sampler itself is constrained* — at each token, only schema-valid continuations are possible (the probability distribution is masked to legal tokens). Wrong-shape output becomes structurally impossible, not merely discouraged. This is the same trick as a compile step: invalid states unrepresentable.

**3. Function/tool signatures (strongest ergonomics):** you declare functions with typed parameters; the model emits arguments conforming to the declaration — which is exactly tool calling (next page). Structured output and tool calls are the same mechanism with different consumers.

## The schema you'll actually write (defensive patterns)

    {
      "type": "object",
      "properties": {
        "severity": {"enum": ["low","medium","high","critical"]},   ← enum, not free string
        "confidence": {"type": "number", "minimum": 0, "maximum": 1},
        "summary": {"type": "string", "maxLength": 500},
        "affected_service": {"type": ["string","null"]}              ← explicit null: "unknown"
        is a VALUE, not silence
      },
      "required": ["severity","summary"],
      "additionalProperties": false                                  ← no surprise fields
    }

Patterns that earn their keep: **enums over strings**, **nullable over optional** (forces the model to express "unknown" rather than omit), **bounds on lengths**, and **`additionalProperties: false`** — schemas are API contracts (the API Design page's rules, verbatim).

## Validation still belongs outside the model

Even with constrained generation, your pipeline should: parse defensively, validate against the schema *again* on receipt, and decide policy on failure (retry with the error message as feedback — often fixes it; fall back to a smaller/safer default; or escalate to a human). The model emits; **the boundary validates** — the same law as "never trust user input", now "never trust model output" either.

## The DevOps mapping

| Structured outputs | Your world |
|---|---|
| Schema-constrained generation | typed API contracts / protobuf |
| Enum/nullable discipline | schema design — make bad states unrepresentable |
| Validate at the boundary | "never trust input" — output is input to the next stage |
| Retry-with-error | failed call + error message back into the loop — reconciler pattern |

## Remember This

1. Structured outputs turn LLMs from chat into components — machine-parseable contracts
2. Reliability ladder: prompt-for-format < schema-constrained sampling < typed tool signatures
3. Constrained generation masks the distribution to legal tokens — invalid JSON becomes impossible
4. Schema defensively: enums, explicit nulls, bounds, no surprise properties
5. Validate at the boundary regardless; retry-with-error as the standard recovery

## One Sentence

Structured outputs impose a machine-readable contract on model generation by constraining sampling to schema-valid tokens, with boundary validation and retry-with-error completing the discipline that turns free-text prediction into a software component.

## Knowledge Check

1. Why does prompt-for-JSON fail eventually, mechanically?
2. Why "nullable" over "optional"? What behavior does each train?
3. Design the retry/fallback policy for a schema-invalid response.

---

**← Previous:** [Context Engineering](context-engineering.md)
**Next:** [Tool Calling](tool-calling.md) →
