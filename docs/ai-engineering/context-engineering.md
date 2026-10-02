# Context Engineering

## What is it?

The prompt is the instruction; **context is everything in the window** — instructions, retrieved documents, conversation history, tool results, examples. **Context engineering** is deciding *what* occupies that finite, paid-for window. Prompting asks "how do I phrase it"; context engineering asks "**what deserves to be in the room at all**".

## Why prompting became insufficient

Prompts worked until products got real: a support assistant needs the ticket, the user's history, three runbook sections, the company policy, last week's incident note... all competing for a window measured in tokens (Tokens page) — while cost scales with every token, and attention quality degrades when the window fills with noise (the Limits page's middle-sag). The phrase "prompt engineering" quietly became "context engineering" the moment the bottleneck moved from *wording* to *allocation*.

## The context budget (think memory management)

    Window = fixed budget (tokens). Allocation decisions:
      instructions (system prompt)     — always resident, small
      task input                      — the actual question/data
      retrieved knowledge              — only what's relevant (RAG page)
      conversation history             — compressed or summarized when long
      tool results                     — truncated, structured
      examples                         — 0-3, only when behavior needs shaping

Every component competes. The engineering moves: **select** (what's relevant), **compress** (summarize history; strip boilerplate from tool output), **prioritize** (critical constraints at start AND end of window — edges get the most attention), **structure** (delimiters, sections — structure steers attention).

## The techniques that define the craft

| Technique | What it does |
|---|---|
| Summarization of history | old turns → a running summary; frees budget |
| Retrieval on demand (RAG) | fetch only relevant chunks at question time (next pages) |
| Tool-result digestion | big outputs → distilled facts before re-entering context |
| Just-in-time loading | don't preload everything "just in case" — load when the task calls for it |
| Context eviction policy | oldest/least-relevant dropped first — an LRU for tokens |

If this table feels like memory hierarchy management (registers → cache → RAM → disk), that's not a coincidence — it *is* that discipline, one level up: **the context window is the working memory; retrieval systems are the disk.**

## The failure modes (what "bad context" looks like)

- **Context poisoning**: irrelevant-but-plausible documents steer the answer wrong (retrieval quality *is* answer quality — RAG page)
- **Lost in the middle**: critical instruction buried mid-window → ignored
- **Context overflow**: truncation silently cuts the wrong end — usually the instructions
- **Cost blowout**: full-history replay per turn — the token bill as incident
- **Injection surface**: every token of external text is steering (Guardrails page)

## The DevOps mapping

| Context engineering | Your world |
|---|---|
| Window = working memory | RAM: fixed, fast, precious |
| Retrieval = disk | fetched on demand, indexed (RAG/vector DB = the index) |
| Summarization | log compaction — keep the signal, drop the volume |
| Prioritize edges | hot partitions — where the access actually is |
| Eviction policy | LRU/cache TTL — your daily bread |

## Remember This

1. Prompt = instructions; context = the whole window; engineering = allocation under budget
2. The shift happened when products needed more knowledge than wording could fix
3. Moves: select, compress, prioritize (edges!), structure, evict — memory management, one level up
4. Retrieval quality is answer quality; irrelevant context actively harms
5. External text in context = steering = an injection surface

## One Sentence

Context engineering is memory management for the model's window — selecting, compressing, and prioritizing what earns a place in finite, paid-for context, with retrieval serving as the disk underneath.

## Knowledge Check

1. Your chatbot replays full history each turn and bills are exploding. Three fixes?
2. Why do critical instructions go at the start *and* end of the window?
3. Map the context budget decisions onto your favorite caching architecture.

---

**← Previous:** [Prompt Engineering](prompt-engineering.md)
**Next:** [Structured Outputs](structured-outputs.md) →
