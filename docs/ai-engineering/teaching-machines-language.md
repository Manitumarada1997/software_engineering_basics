# Teaching Machines Language

## What is it?

Every approach humans tried to make computers process language before modern LLMs — rules, statistics, n-grams — and the specific wall each one hit. This page is the graveyard that explains why tokens and embeddings had to be invented.

## Attempt 1: rules again (1950s–1980s)

Grammar rules + dictionaries: "IF sentence matches pattern THEN meaning." ELIZA (1966) fooled people with pattern-matching therapy responses. The wall is the familiar one — natural language has millions of patterns, constant invention, and meaning that ignores grammar ("Time flies like an arrow; fruit flies like a banana").

## Attempt 2: counting — n-grams (1990s–2000s)

New idea: stop modeling grammar; **count what actually follows what** in a corpus. An **n-gram** is a sequence of n words; the model learns "after 'New York' comes 'City' 40% of the time, 'State' 20%..." — pure probability (Probability page) from counting.

    "The cat sat on the ___"   ->   mat 62% · floor 15% · chair 8% · roof 0.1% · ...

This genuinely generated text, powered autocomplete, and beat rules at translation for a decade. Its three fatal walls:

1. **Memory explosion**: counts for every 5-word combination in existence — more data, exponentially more counts
2. **Exact-match blindness**: "The cat sat on the mat" helps not at all with "The feline rested on the rug" — *words are treated as unrelated symbols*
3. **No meaning**: it knows sequences, not what a cat *is* — long, coherent text was impossible

## The wall, stated precisely

**Computers store words as unrelated ID numbers.** "cat" = 1043, "feline" = 8921 — nothing about 1043 is close to 8921. Every human intuition of "similar meaning" is invisible to the machine.

That is *the* problem the next ideas solve, and it is worth saying exactly: **a computer needs meaning represented as numbers where similarity is a measurable distance**. Not lists of synonyms (rules again), but a geometry.

## Why this section's story matters to you

The n-gram → embedding → RNN → attention → transformer sequence is *exactly* the course's evolution pattern: each solution's specific limitation is the next solution's founding problem. When an LLM fails at something today, you will debug it by knowing which layer of this history is responsible — tokenization? retrieval? attention window? The lineage is the debugging map.

## What came next

Two inventions split the problem: **embeddings** (turn words into meaningful coordinates — next page) and **sequence models** (handle order and context — the page after). Their combination, plus attention, becomes the transformer.

## Remember This

1. Rules for language died on pattern explosion; ELIZA was pattern matching, not understanding
2. N-grams = counting co-occurrence: real generation, but exact-match-blind and memory-hungry
3. The core wall: words as unrelated IDs — similarity and meaning invisible to the machine
4. Needed: meaning as numbers where similarity is measurable distance (a geometry)
5. The lineage (n-gram → embedding → RNN → attention → transformer) is your debugging map

## One Sentence

Every pre-neural approach to language died on the same wall — computers saw words as unrelated symbols — and that wall is what embeddings were invented to break.

## Knowledge Check

1. Why does an n-gram model waste everything it knows about "cat" when it meets "feline"?
2. Why did counting (statistics) beat grammar rules for translation?
3. State the exact problem embeddings must solve, without using the word "vector".

---

**← Previous:** [The Deep Learning Era](deep-learning-era.md)
**Next:** [Word Embeddings](word-embeddings.md) →
