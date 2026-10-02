# Training an LLM (and the offline question)

## What is it?

Training an LLM = running the Learning page's loop at planetary scale: feed a huge slice of the internet's text through a Transformer, and for every position, check "did the model predict the actual next token?" — loss, gradients, nudge 70 billion weights. Repeat for *trillions* of tokens. This is **pretraining** — and it happens **once**, at enormous cost, producing the frozen weights you later download.

## Pretraining, concretely

| Aspect | Reality |
|---|---|
| Data | web crawl (filtered), books, code, papers — trillions of tokens |
| The task | literally one thing: predict the next token, everywhere, forever |
| Compute | thousands of GPUs for weeks-to-months; $1M–$100M+ for frontier models |
| Output | weights: a file (GBs to hundreds of GBs) that *is* the model |

Nobody labels "this is grammar", "this is physics". The task generates its own answers — the actual next token of real text is the label. That's why pretraining scales: the internet is the self-grading textbook.

## The critical concept: TRAINING vs INFERENCE

    TRAINING (once, offline, expensive)
       huge dataset -> forward pass -> compare prediction vs actual next token
       -> loss -> update weights (gradient descent) -> repeat trillions of times
       Result: a frozen model file

    INFERENCE (every use, online, cheap)
       your prompt -> forward pass ONLY -> next token -> feed back -> next token
       -> answer. NO weight updates. NO learning happening.

Inference is read-only arithmetic with frozen numbers. Two consequences that unlock the next section:

1. **An offline model works** — because its "knowledge" is baked into the weights during training. No internet needed: the internet was consumed *beforehand*, compressed into parameters. (Fresh or private data is exactly what it *cannot* have — that's the RAG story.)
2. **Model knowledge vs internet access vs retrieval** — three different things:
   - *Model knowledge*: whatever the training data contained, frozen at training time
   - *Internet access*: none, unless you build it (tools, later)
   - *Retrieval*: fetching documents at question-time and putting them in the context window (RAG, later)

**"Where does an offline LLM's knowledge come from?"** — From its training data, compressed into weights, the way your Kubernetes knowledge comes from years of incidents, not from a live connection to a cluster.

## Fine-tuning, briefly (full page later)

Pretraining makes a *text predictor*. Additional, cheaper training phases specialize it: **supervised fine-tuning** (example dialogues), **RLHF** (preference feedback). Details on the Alignment page; the point here: fine-tuning = more training, on much less data, cheaply adjusting an existing model — never rebuilding one.

## The DevOps mapping

| Training | Your world |
|---|---|
| Pretraining run | a one-time migration producing the golden artifact |
| Weights file | the immutable image — version it, checksum it |
| Trillions of repeats | embarrassingly parallel batch jobs |
| Inference = forward-only | serving the artifact; no writes to the image |
| Train/inference split | build time vs run time — the oldest distinction you own |

## What problem did this create?

Pretraining produces a brilliant *predictor of internet text* — ask it a question and it may answer, or may continue like a conspiracy forum, or answer as a pirate. It has no concept of "being a helpful assistant". Turning predictors into assistants is **alignment** — next page.

## Remember This

1. Pretraining = next-token prediction over internet-scale text, once, at huge cost
2. The next token *is* the label — self-grading, which is why it scales
3. Training updates weights (once); inference is read-only frozen arithmetic (always)
4. Offline models "know" because training compressed the data into weights
5. Knowledge is frozen: freshness and privacy are exactly what weights can't have — RAG's reason to exist

## One Sentence

Training an LLM is the one-time, expensive process of compressing trillions of text tokens into frozen weights by predicting each next token, while inference is the cheap, read-only loop that reuses those weights forever without learning anything new.

## Knowledge Check

1. Why is pretraining self-grading, and why does that matter for scale?
2. Explain to a colleague why an offline model "knows" things without internet.
3. Name what weights can never contain (two things), and what technology supplies each.

---

**← Previous:** [How LLMs Generate Text](how-llms-generate.md)
**Next:** [Alignment](alignment.md) →
