# The Limits of LLMs

## Why this page exists

Every LLM failure you will debug in production is one of about six structural limits. Each has a *mechanism* (from pages you've read) and a *remedy* (from pages ahead). This page is your diagnostic map.

## The six structural limits

**1. Hallucination — confident fabrication.**
Mechanism: the model *must* output a plausible next token (Probability page) — "I don't know" is a token choice it was never systematically rewarded for, and plausibility is what the weights encode. Confident tone ≠ truth (calibration, page 2).
Remedy: grounding (RAG), citations, retrieval-augmented verification, evals that catch it. Never fixable by prompting alone.

**2. Frozen knowledge — a training-date cutoff.**
Mechanism: weights were frozen at training time (Training page); nothing learned since exists in the model.
Remedy: RAG for fresh documents; tools for live data. "What's your knowledge cutoff?" is an API question about a filesystem image date.

**3. No private data — it never saw your company.**
Mechanism: your runbooks, tickets, and code aren't in the training corpus.
Remedy: RAG (embed your docs) or fine-tuning (expensive, and knowledge edits poorly — the Model page's smeared-facts law).

**4. Context limits — the window is finite and attention degrades.**
Mechanism: the window (Tokens page) holds finite tokens; beyond the limit, nothing exists; within very long contexts, recall of the middle sags ("lost in the middle" — attention is strongest at the edges). There is no persistent memory between conversations (Memory page later).
Remedy: context engineering — select, compress, prioritize; external memory stores.

**5. No actions — an LLM alone cannot query a DB or call an API.**
Mechanism: it predicts tokens. Full stop. Any "action" is an illusion unless *you* build the plumbing.
Remedy: tool calling (the page after next) — the model outputs a structured request, *your code* executes it.

**6. Reasoning brittleness — arithmetic, counting, multi-step slips.**
Mechanism: prediction is pattern completion, not symbolic execution; letter-counting breaks on tokenization (Tokens page); long logic chains drift because each step is sampled.
Remedy: structured outputs + code execution for exact work ("write a function, don't compute in your head"); reasoning-effort models; step-by-step prompting; evals.

## The master table (print this)

| Limit | Mechanism | Remedy | Remedy's page |
|---|---|---|---|
| Hallucination | must-produce-plausible-token | grounding, citations, evals | RAG, Evals |
| Frozen knowledge | weights frozen at cutoff | retrieval | RAG |
| No private data | never in training corpus | retrieval > fine-tuning | RAG |
| Context finite | window budget, middle sag | context engineering | Context |
| No actions | predicts tokens only | tool calling | Tools |
| Reasoning slips | sampling ≠ execution | code execution, schemas | Structured outputs |

## The reframe that makes you an AI engineer

Beginners ask: "how do I make the model smarter?" Engineers ask: **"which structural limit am I hitting, and which surrounding system component compensates for it?"** The model is fixed at deploy time; the *system around it* is your entire leverage. That system — retrieval, tools, guards, memory, evals — is what the rest of this track builds.

## The DevOps mapping

You have debugged "the network is slow" into DNS/MTT/queueing; here you debug "the AI is wrong" into hallucination/cutoff/context/tooling. Same discipline: name the mechanism, apply the remedy, monitor for regression.

## Remember This

1. Six limits: hallucination, frozen knowledge, no private data, finite context, no actions, reasoning slips
2. Every limit has a mechanism (pages you've read) and a system remedy (pages ahead)
3. Confidence is never evidence — calibration law, applied
4. The model is frozen at deploy; the system around it is your leverage
5. "Make the model smarter" is the wrong question; "which limit, which component" is right

## One Sentence

LLMs fail in six structural ways — fabrication, stale frozen knowledge, no private data, finite context, inability to act, brittle reasoning — each inherent to next-token prediction and each compensated for by system components rather than by the model itself.

## Knowledge Check

1. Match: citing a nonexistent paper / unaware of yesterday's incident / forgetting turn 1 of a long chat / claiming it "ran" the script — which limit each?
2. Why can fine-tuning not fix "knowledge cutoff" sustainably?
3. Why is "prompt harder" never the remedy column for limits 4 and 5?

---

**← Previous:** [Alignment](alignment.md)
**Next:** [Open & Closed Models](open-and-closed-models.md) →
