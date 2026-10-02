# From Calculators to Learning Machines

## What is this about?

Before we can explain what an LLM is, we must answer a stranger question: **how did we get from computers that follow fixed instructions to computers that learn?** This page tells that story — because every AI term you will meet (model, weights, training) exists to solve a problem that appeared at one specific moment in this story.

## The problem: some things cannot be written as rules

A computer is a machine that follows instructions perfectly. This is its superpower — and for seventy years it worked brilliantly:

```text
Instruction: "if order_total > 100 then discount 10%"
Result: correct, every time, forever.
```

But some human tasks resist instructions. Try writing the *rules* for:

- Recognizing a cat in a photo (is it the ears? the fur? what about a cartoon cat?)
- Understanding that "the movie was so bad it was good" is *positive-ish*
- Translating a joke — the rules of humor fill libraries and fail anyway

**The rule-writer's wall:** for these tasks, humans can't explain their own knowledge. You recognize your friend's face instantly, but you cannot write down *how*. If the knowledge can't be written down, it can't be programmed.

```text
Traditional programming:
    RULES + DATA → answers

The wall: no rules exist for "catness", "fluency", "sarcasm".
```

## The old attempts — and exactly why each failed

**1. More rules (1950s–1980s: symbolic AI / expert systems).** Knowledge engineers interviewed experts and encoded thousands of IF/THEN rules ("MYCIN" diagnosed infections). It worked in narrow, tidy domains — then hit the wall of the real world: rules exploded in number, contradicted each other, and snapped at every edge case. Maintaining ten thousand rules is config-drift hell (you know this disease) with no test suite.

**2. Statistics without understanding (the same era).** Systems counted word frequencies and guessed. Cheaper, more robust — but shallow: they could predict "the" follows "a", and almost nothing else.

**3. Neural networks, attempt one (1958: the Perceptron).** A different idea: *stop writing rules — let the machine adjust itself until it produces correct answers.* We'll explain this machine on later pages. In 1958 it failed for a reason worth remembering: nobody knew how to *train* networks of more than one layer, and one layer was too weak. AI winters followed — funding froze twice (1974, 1987) when promises outran results.

## The new idea: learning from examples

The breakthrough — decades in arriving — was a change of direction, not a new machine:

```text
Traditional:  RULES + DATA          → answers
Machine learning:  DATA + ANSWERS   → RULES (the machine writes them, as numbers)
```

You don't explain what a cat is. You show 10,000 labeled photos ("cat", "not cat") and a procedure adjusts the machine's internal numbers until its answers match. The "rules" it finds are mathematical patterns humans never managed to articulate — stored as billions of adjustable numbers.

If you remember one sentence from this page:

> **A "model" is a machine whose behavior comes from numbers that were adjusted to fit examples, rather than from instructions a human wrote.**

Every remaining term — weights, parameters, training, inference — is vocabulary for this one idea.

## Where the connection to your world begins

You already operate the machine-learning *pattern*, just under a different name:

| Machine learning | DevOps equivalent |
|---|---|
| Examples (data) | telemetry / metrics |
| Training procedure | a pipeline that tunes a system |
| The trained model (numbers) | the artifact your pipeline produces |
| Inference (using it) | running the artifact in production |
| Bad data → bad model | garbage in, garbage out (same law) |

The entire course you completed was about *deterministic* systems: same input → same output → tests can assert. The single most important mental shift of this track: **AI systems are adjusted-to-fit systems.** Their behavior is grown from data, like a garden — not assembled from specs, like a bridge. That difference will explain everything strange that comes later: why they hallucinate, why they can't be unit-tested in the classic sense, and why "evals" exist.

## Remember This

1. Some knowledge (faces, language) can't be written as rules — the rule-writer's wall
2. Traditional programming: rules + data → answers. ML: data + answers → rules
3. The 1958 idea failed on training; AI winters followed; the *idea* waited for data + compute
4. A model = behavior from adjusted numbers, not written instructions
5. Grown from data (garden), not assembled from spec (bridge) — the track's master mental model

## One Sentence

Computers learned to "learn" when engineers stopped trying to write rules nobody could articulate and started showing machines examples until the machines wrote the rules themselves, as numbers.

## Knowledge Check

1. Write the two-line difference between traditional programming and machine learning.
2. Why did expert systems fail at "catness" when they succeeded at infection diagnosis?
3. Why does "the model is an artifact" make sense to a DevOps engineer?
4. What breaks about classic unit testing when behavior comes from data instead of instructions?

---

**← Previous:** [Track Overview](index.md)
**Next:** [Probability & Prediction](probability-and-prediction.md) →
