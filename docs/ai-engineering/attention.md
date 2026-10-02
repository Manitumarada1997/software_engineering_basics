# Attention

## What is it?

**Attention** lets a model, when processing one word, look *directly* at every other word in the input and decide — with numbers — which ones matter for the job at hand. No chains of rewrites, no fading memory: every word can consult every other word, at any distance, in one step.

## The problem it solves, concretely

    "The dog chased the ball because it was excited."
                                        ^^
    What is "it"? The dog. A human resolves this instantly; an LSTM must carry "dog" through seven rewrites to reach "it" — usually degraded (vanishing memory).

Attention's answer: when processing "it", directly *ask every previous word how relevant it is* — score them, and blend their information by score:

    "it" attends to:  dog(0.7) · ball(0.2) · chased(0.05) · the(0.01)...
    meaning of "it"  ≈ mostly dog, a bit of ball

The long-range wall dissolves because there is no *range* — distance costs nothing.

## Query, Key, Value — the library metaphor (the real vocabulary)

Every word publishes three vectors (small learned transformations of its embedding):

| Vector | Library metaphor | Role |
|---|---|---|
| **Query** (Q) | "I'm looking for books about X" | what this word *needs* |
| **Key** (K) | the label on each book's spine | what each word *offers* |
| **Value** (V) | the book's actual content | what gets blended in if selected |

Processing "it": its **query** is compared against every word's **key** → match scores → scores become weights → weighted blend of everyone's **values** becomes the new, context-aware meaning of "it". The scores are literally dot products — cosine-similarity's cousin (Embeddings page) doing retrieval *inside* the network.

Note the deep echo: attention *is* a micro search engine inside the model — query against keys, blend the values. When you later build RAG (search *outside* the model), you will recognize the same Q/K/V retrieval logic at application scale. Nature rhymes.

## Self-attention & multi-head

**Self**-attention: the sequence attends to *itself* (words asking words) — no external input needed.
**Multi**-head: run several attentions in parallel — one head tracks syntax ("it"→"dog"), another tracks sentiment, another tracks numbers. Each head learns its own kind of relationship; their findings combine. Eight heads means eight simultaneous relationship-trackers per layer, stacked dozens of layers deep: relationships of relationships.

## The one remaining problem attention didn't solve

Attention connects everything to everything — but connections alone don't process anything, and critically: **attention is order-blind**. The scores for "dog bites man" and "man bites dog" would be identical — every word attends equally to every other! Something must tell the model what order words are in. That fix — plus packaging attention into a full, *parallel* architecture — is the Transformer, next page: the invention that turned this mechanism into the machine that became the LLM.

## The DevOps mapping

| Attention | Your world |
|---|---|
| Q against K | service discovery: "who can answer this?" |
| Weighted blend of V | an aggregator merging scored sources |
| Multi-head | parallel specialized workers on the same request |
| Attention inside the model | a micro service-mesh routing context |

## Remember This

1. Attention = direct, scored connections between any two words, at any distance, one step
2. Query (need) vs Key (offer) match scores how much each Value blends in
3. Self-attention: the sequence attends to itself; multi-head: several relationship-types in parallel
4. It kills vanishing memory — distance stops costing anything
5. It's order-blind alone — positional information must be added (the Transformer's job)
6. Q/K/V is retrieval logic — RAG later reuses the same pattern outside the model

## One Sentence

Attention lets every word directly query every other word and blend what matters most by relevance score — dissolving the long-range memory problem that sequence models died on.

## Knowledge Check

1. Trace "it was excited" with Q/K/V: who queries, who scores high, what blends in?
2. Why is attention order-blind, and what must fix it?
3. Why is attention's cost independent of distance but quadratic in sequence length (hint: every word attends to every word)?

---

**← Previous:** [RNNs & LSTMs](rnns-and-lstms.md)
**Next:** [Transformers](transformers.md) →
