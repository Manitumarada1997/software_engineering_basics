# Prompt Engineering

## What is it?

A **prompt** is everything you put in the context window to steer the model: instructions, role, examples, constraints. Prompt engineering is designing that input deliberately — and now that you know the mechanism (next-token prediction over your context), it stops being folklore:

**Why prompts work**: the model continues text in the style and direction of its context. A well-designed prompt makes the *desired answer the most plausible continuation*. You're not commanding a mind; you're shaping the probability landscape.

## What problem was it solving?

Early LLM products failed on steering: users typed vague questions, got vague completions; developers got inconsistent behavior between runs. Prompting emerged as the first control surface — the "configuration layer" over a model that can't be configured any other way at deploy time.

## The anatomy of a strong prompt (the components, named)

    [ROLE]       You are a Kubernetes incident summarizer for ShopEasy's platform team.
    [TASK]       Summarize the incident timeline below for the on-call handover.
    [CONSTRAINTS] Max 5 bullets. Include: trigger, blast radius, current status.
                  No speculation; mark unknowns as UNKNOWN.
    [CONTEXT]    --- timeline: 14:00 alert fired ... 14:32 mitigated ---
    [FORMAT]     Output as markdown bullets, most severe first.

Note what this really is: **requirement specification for a non-deterministic executor** — the Requirements page's discipline (unambiguous, testable) reborn for prompts.

**Few-shot** = include 2–3 input→output examples in the prompt: the model continues the *pattern* (it's literally pattern completion). **Zero-shot** = instructions only. **Chain-of-thought** = ask for reasoning steps before the answer — arithmetic/planning improves because each intermediate token becomes context that constrains the next (externalized working memory).

## The limitations (what prompting alone cannot do)

This list is the Limits page, restated as engineering constraints — and each limitation's remedy is the *next pages*:

| Prompting can't | Because | Remedy |
|---|---|---|
| inject knowledge the weights don't have | frozen knowledge | RAG |
| fit unlimited context | window budget | context engineering |
| reliably return parseable JSON | free-text sampling | structured outputs |
| perform actions | tokens only | tool calling |
| guarantee safety | shaping not security | guardrails |

**Prompt ≠ configuration theater**: an over-stuffed prompt with contradictory rules degrades output (attention diluted over noise). Prompting is engineering: versioned, tested, measured — or it's superstition.

## Prompts as code (the discipline)

- **Versioned in Git** with changelogs — a prompt is a dependency of your system
- **Tested** against golden sets (Evals page) before promotion
- **Parameterized** as templates (variables for context; fixed instruction scaffolding)
- **Traced**: every production call logs prompt version + model version (Observability page)
- **Prompt injection** (the security flip side): any text in context is *steering* — including text you didn't write (retrieved docs, user input). That's a security page of its own (Guardrails).

## The DevOps mapping

| Prompting | Your world |
|---|---|
| Prompt = config for a frozen artifact | feature flags over an immutable image |
| Prompt versioning | dependency versioning — upgrades need regression tests |
| Few-shot examples | golden-path templates: "this is the pattern" |
| Contradictory instructions | config conflicts — last-writer-wins chaos |

## Remember This

1. Prompts shape the probability landscape: desired answer = most plausible continuation
2. Components: role, task, constraints, context, format — requirement spec for a probabilistic executor
3. Few-shot = pattern continuation; chain-of-thought = externalized working memory
4. Prompting's ceiling is the Limits page — each gap has a system remedy (next pages)
5. Prompts are code: versioned, tested, traced — never folklore

## One Sentence

Prompt engineering is requirement specification for a non-deterministic executor — shaping context so the desired answer becomes the most plausible continuation, with versioning and testing as the discipline that separates engineering from superstition.

## Knowledge Check

1. Explain why few-shot works strictly in terms of pattern completion.
2. Rewrite a vague prompt ("summarize this") into the five components.
3. Why is "we fixed it in the prompt" a supply-chain risk for your product?

---

**← Previous:** [Serving at Scale](serving-at-scale.md)
**Next:** [Context Engineering](context-engineering.md) →
