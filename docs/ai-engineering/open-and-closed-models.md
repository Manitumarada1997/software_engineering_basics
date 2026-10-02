# Open & Closed Models

## What is it?

Two ways models reach you:

| | Closed ("API models") | Open-weight ("local models") |
|---|---|---|
| What you get | answers over an API | the weights file itself |
| Examples | GPT-4/5, Claude, Gemini | Llama, Mistral, Qwen, Gemma, DeepSeek |
| Where it runs | the provider's fleet | your laptop / your cluster |
| Cost model | per token | your hardware + electricity |
| Knowledge/quality | frontier, usually | one step behind, closing |
| Privacy | data leaves your perimeter | data never leaves |
| Change control | provider changes it under you (versioning!) | you pin the file forever |

"Open-weight" is the precise term: the *parameters* are published; the training data and pipeline usually aren't (not "open source" in the software-freedom sense). **Distillation** (defined properly in the Fine-tuning page) is how smaller open models learn from bigger ones — much of the open ecosystem is distilled frontier behavior running on your hardware.

## Why both exist (the forces)

- **Closed** won capability: scaling laws' budget line — only a few orgs can spend $100M per training run, and they monetize via API
- **Open** won everything else: privacy (regulated data can't leave), sovereignty (no provider dependency), cost at scale (tokens at fleet volume), latency (on-device), and research (inspectable weights)
- The gap: open models trail the frontier by roughly a generation — permanently, because they're often distilled from it — but "last generation's frontier" keeps exceeding most workloads' needs

## The decision table (the page's practical output)

| Situation | Choice |
|---|---|
| Regulated/secret data (logs, PII, source) | open, local — the data must not leave |
| Frontier capability (hard reasoning, agents) | closed API |
| High volume, simple tasks (classify, extract) | open or small — unit economics |
| Offline/embedded/edge | open, quantized (next pages) |
| Prototype | closed — fastest iteration |
| Production, fixed behavior, auditability | open, pinned, versioned |

Hybrid is the norm: prototype on frontier APIs, distill/route production to smaller models (the Cost page's routing pattern).

## Your unfair advantage

Running models *is* infrastructure engineering: memory, throughput, scheduling, observability — your six years. The next section is where that advantage compounds: how inference actually consumes hardware, what quantization trades, and how to run production-grade local AI.

## Remember This

1. Closed = answers via API; open-weight = the numbers themselves — you run them
2. Closed wins capability; open wins privacy, sovereignty, unit cost, latency, pinning
3. Open trails by a generation — often distilled — which still exceeds most needs
4. Decision by data sensitivity, capability need, volume, and change control
5. Hybrid routing (frontier for hard, small/local for bulk) is the standard production shape

## One Sentence

Closed models rent you frontier capability by the token while open-weight models hand you the numbers themselves — and the choice between them is a data-sensitivity, capability, volume, and change-control decision, not an ideology.

## Knowledge Check

1. Why is "open-weight" not "open source"?
2. Your org wants AI on support tickets containing PII. Walk the decision.
3. Why does the open ecosystem stay one generation behind — and why is that usually fine?

---

**← Previous:** [The Limits of LLMs](limits-of-llms.md)
**Next:** [How Inference Works](how-inference-works.md) →
