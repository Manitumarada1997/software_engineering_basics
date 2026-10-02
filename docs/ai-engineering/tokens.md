# Tokens

## What is a token?

A **token** is the piece of text an LLM actually reads and writes — usually a common word, a chunk of a word, or punctuation. An LLM cannot read letters or words directly; it only understands sequences of token IDs (numbers).

    "I love Kubernetes"
      -> ["I", " love", " Kuber", "net", "es"]      (a typical tokenizer's cut)
      -> [40, 1842, 38427, 1412, 251]               (token IDs — the vocabulary's index)

## Why not words? Why not characters?

The two obvious options both fail:

- **Whole words**: the vocabulary explodes (every language, every name, every typo, every version string). And the model can never handle a word it never saw — one miss, total failure.
- **Characters**: tiny vocabulary, but a sentence becomes hundreds of steps — attention over long spans degrades, and learning that "K-u-b-e-r" is a unit wastes capacity on spelling.

**Subword tokenization** (BPE — byte-pair encoding) is the compromise, learned from data: start from characters, repeatedly merge the most frequent pairs until you have ~50k–200k units. Common words become single tokens (" love"); rare words split into reusable pieces (" Kuber" + "net" + "es" — and now " Kubernetes"-ish spellings and " Kubernet" reuse those pieces for free).

## What tokens mean in practice (the engineer's four consequences)

**1. Context windows are counted in tokens.** A "128k context" model holds ~128,000 tokens ≈ a small book. Every prompt, document, and conversation chunk competes for that budget — token budgeting is memory budgeting.

**2. Cost is counted in tokens.** Providers bill input tokens + output tokens separately (output is pricier — generation is sequential work). A prompt with a whole PDF pasted in costs that PDF's tokens *per request*. This is the FinOps line item of every AI feature.

**3. Speed is counted in tokens.** Generation emits tokens one at a time (next page), so "tokens per second" is your throughput metric; a response feels fast at 30+ tokens/second, roughly reading speed.

**4. Models have token-shaped blind spots.** Counting letters ("how many r's in strawberry") fails because the model never sees letters — it sees one token "strawberry". Weird tokenization boundaries cause arithmetic slips and exotic-word errors. "ChatGPT reads tokens, not human concepts" — most surprising LLM failures are tokenizer artifacts.

## The vocabulary, defined

The **vocabulary** is the fixed table (built at training time) mapping token text to IDs — ~100k entries, frozen into the model. The **embedding table** (Embeddings page) is exactly this: one learned vector per vocabulary entry. Input token ID 38427 → row 38427 of the table → its vector enters the Transformer. That lookup *is* how text becomes numbers.

## The DevOps mapping

| Tokens | Your world |
|---|---|
| Token budget | memory budget — everything competes for the window |
| Input/output token pricing | egress vs compute pricing — different costs, same bill anxiety |
| Tokens/second | throughput/latency SLO metrics |
| Vocabulary | a fixed enumeration — like a protocol's schema |

## What came next

You now know what the model reads. The next page runs the full loop — from your question as tokens to the generated answer, one token at a time — and demystifies why predicting pieces of text produces something that appears to think.

## Remember This

1. Tokens = subword pieces; IDs into a frozen vocabulary; the model never sees letters or words
2. Subwords solve the vocabulary-vs-length tradeoff (BPE: frequent merges, learned from data)
3. Token counts govern context, cost, and speed — the budget unit of AI systems
4. Letter-counting failures are tokenizer artifacts, not reasoning failures
5. Embedding table = one vector per vocabulary entry; the lookup is how text becomes numbers

## One Sentence

Tokens are the subword pieces and numeric IDs through which an LLM perceives all text — making the token the universal unit of context, cost, and speed.

## Knowledge Check

1. Why does "strawberry" letter-counting break? Show the tokenization.
2. Why does BPE handle a never-before-seen product name without failing?
3. Your feature pastes a 40-page PDF per request. What three budgets does that hit?

---

**← Previous:** [Scaling Laws](scaling-laws.md)
**Next:** [How LLMs Generate Text](how-llms-generate.md) →
