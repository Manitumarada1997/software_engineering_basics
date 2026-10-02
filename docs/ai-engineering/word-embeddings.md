# Word Embeddings

## What is it?

An **embedding** is a list of numbers (a **vector**) assigned to a word, built so that **words with similar meaning get similar coordinates**. "cat" might become `[0.2, 1.3, -0.7, ...]` and "feline" `[0.3, 1.1, -0.6, ...]` — nearby in that space. Meaning becomes geometry; similarity becomes measurable distance.

## Why does it exist?

To break the previous page's wall: words as unrelated IDs. The brilliant insight (Word2Vec, 2013): **you can learn these coordinates from plain text, with no labels**, because of a linguistic truth: *words appearing in similar contexts have similar meanings* ("cat" and "feline" both fill the "___ slept on the windowsill" slot). A small neural network is trained to predict a word from its neighbors — and the weights it learns *along the way* turn out to be coordinates with meaning.

## The famous proof: king − man + woman ≈ queen

Embeddings don't just place similar words together — *relationships become directions*:

    king - man + woman  ≈  queen      (gender is a direction)
    paris - france + italy ≈ rome     (capital-of is a direction)

Nobody programmed those directions. They emerged from the geometry of usage — the strongest single piece of evidence that "meaning as coordinates" is real.

## How similarity is measured (the one formula of this track)

Each word is an arrow from the origin; **cosine similarity** measures the *angle* between arrows:

    1.0  same direction  (identical meaning-ish)
    0.0  perpendicular   (unrelated)
    -1.0 opposite        (antonyms — though in practice antonyms often sit close: same contexts!)

You will meet this exact computation again — in *every RAG system*, where "find relevant documents" means "find vectors near the question's vector."

## Embeddings beyond words

The idea generalizes: sentences → one vector; documents → one vector; *your entire runbook library → vectors*. Anything embedded into the same space can be compared by distance: search, clustering, deduplication, recommendations. Vector databases (later, in the RAG page) are indexes for exactly this "nearest arrow" query at scale.

## The limitation that forced the next step

Word2Vec gives **one fixed vector per word**. But "bank" means different things in "river bank" and "bank account" — one coordinate per word can't capture context-sensitivity, and long passages can't be squeezed into word-level coordinates. Language needed representations that *change with context* — sequence models. Next page: RNNs and LSTMs, the first serious attempt.

## The DevOps mapping

| Embeddings | Your world |
|---|---|
| Vector space | a metric space — like latency distributions you compare |
| Cosine similarity | the "match score" — alert correlation by shape, not string |
| Context-prediction training | learning structure from telemetry without labels |
| Vector DB (preview) | an index for nearest-neighbor queries — like a specialized cache |

## Remember This

1. Embedding = a word's coordinates; similar meaning = nearby points; built from contexts, unlabeled
2. Relationships are directions: king − man + woman ≈ queen — structure emerged, wasn't coded
3. Cosine similarity = angle between arrows; the workhorse of every RAG system you'll build
4. Anything can be embedded: sentences, docs, runbooks — one comparable space
5. Limitation: one fixed vector per word — context-blind — sequence models come next

## One Sentence

Embeddings turn words into coordinates learned from context so that meaning becomes measurable distance — the invention that finally gave computers a geometry of language.

## Knowledge Check

1. Explain why "learned from contexts, no labels" made embeddings scale where expert systems didn't.
2. Why do antonyms often embed close together? What does that teach about "similarity"?
3. Where will you personally meet cosine similarity in production this decade?

---

**← Previous:** [Teaching Machines Language](teaching-machines-language.md)
**Next:** [RNNs & LSTMs](rnns-and-lstms.md) →
