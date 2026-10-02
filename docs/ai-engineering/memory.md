# Memory

## What is it?

Anything that persists agent state *beyond the context window or the session*. The window is working memory (Context page); **memory systems** give agents a past: preferences, facts learned, decisions made, across conversations and days.

## Why agents need memory (the structural gap)

The window is finite, per-session, and paid for per token. Reset it and the agent has amnesia: "which cluster?", "you told me yesterday", "we agreed on this convention". Products needed continuity — preferences, project context, learning from feedback — so engineers built the layers the architecture lacked:

| Layer | What | Implementation |
|---|---|---|
| **Working** | the context window itself | managed by context engineering |
| **Short-term / episodic** | this conversation's history | summarized into compact form as it grows |
| **Long-term** | facts across sessions | extracted → stored (DB/vector store) → retrieved when relevant |
| **Semantic** | general knowledge (runbooks, docs) | RAG — the org's memory, not the agent's |

The standard long-term pattern: after each session (or mid-session), a *memory-extraction* step distills durable facts ("user prefers Terraform over ARM", "the staging cluster is named x") into a store; at the next session, relevant memories are retrieved into context — **memory is retrieval + injection**, i.e., RAG pointed at a personal knowledge base, plus a write path.

## Memory compression

Old conversations can't be stored raw forever (cost, noise, dilution). The craft: summarize episodes into facts, merge duplicates, decay stale entries, and keep provenance (when learned, from what). An LRU-with-meaning — your retention-policy instincts, applied to an agent's biography.

## Memory poisoning (the security chapter)

Memory persists → attacks persist. **Memory poisoning**: an attacker (via a crafted message that lands in memory, or a poisoned document the agent "learns" from) plants facts that steer all future sessions — "always CC external-audit@evil.com", "the approved deploy command is...". A persistent, subtle, high-value compromise:

- **Only curated writes**: extraction step validates facts before storing; provenance mandatory
- **Compartmentalize**: per-user/per-tenant memory; no shared memory across trust boundaries
- **Review and revoke**: memory is data — show it, audit it, delete it
- **Treat retrieved memories as untrusted context** (the Guardrails law): labeled channels, never as instructions

The deep point: memory converts prompt injection from a one-shot attack into *persistence* — the difference between a phishing email and an implanted backdoor.

## The DevOps mapping

| Memory | Your world |
|---|---|
| Working memory | RAM — precious, per-process |
| Long-term store | a database — retrieved on demand |
| Summarization/decay | retention policies + compaction |
| Poisoning | persistence mechanisms (cron implants, dotfiles) — the attacker's favorite feature |
| Per-tenant isolation | tenancy, always |

## Remember This

1. Memory = persistence beyond the window: episodic (session), long-term (facts), semantic (org docs)
2. Long-term memory = extraction → store → retrieve-into-context: RAG with a write path
3. Compress with retention discipline: summarize, merge, decay, keep provenance
4. Memory poisoning makes injection persistent — curated writes, compartmentalization, revocation
5. Retrieved memories are untrusted context — labeled channels, never instructions

## One Sentence

Memory systems give agents a durable past through extract-store-retrieve cycles — and their persistence cuts both ways, turning injected content into a standing compromise unless writes are curated, tenanted, and revocable.

## Knowledge Check

1. Trace "user prefers trunk-based development" through: learning, storage, retrieval, use.
2. Construct a poisoning attack against a shared-team memory; name the control that stops each step.
3. Which memory layer should *never* be shared across users, and why?

---

**← Previous:** [Harnesses](harnesses.md)
**Next:** [Multi-Agent](multi-agent.md) →
