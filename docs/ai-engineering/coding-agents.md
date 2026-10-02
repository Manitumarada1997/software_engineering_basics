# Coding Agents

## What is it?

The evolution of AI-assisted development — each stage born from its predecessor's ceiling:

```text
Autocomplete (2019)     — next-line prediction; fast, zero context
   ↓ couldn't see intent beyond the line
Chat assistant (2022)   — ask about code; you copy-paste both ways
   ↓ context lived in your clipboard, not the tool
Code assistant (2023)   — reads the repo, edits files; you drive every step
   ↓ multi-step tasks were you-in-the-loop as the loop
Repo-aware agents (2024)— plans, edits across files, runs tests, iterates on failures
   ↓ you still assign each task
Coding agents (2025+)   — given an issue: explores repo, plans, implements, tests, opens PR
```

## Why each stage emerged (the pattern, once more)

Every stage's ceiling was the next stage's trigger — the same evolution logic as the whole track. The current stage is an **agent in a harness** (Harness page) whose tools are: repo read, file edit, shell, test runner, search. The loop (Loops page): explore → plan → implement → run tests → read failures → fix → repeat → PR. The model composes; *the harness* enforces.

## The harness around a coding agent (the real product)

| Component | In a coding agent |
|---|---|
| Context management | repo maps, selective file loading — the repo never fits the window; selection is the game |
| Tool registry | read/edit/shell/test, scoped |
| Verification | **tests as the eval** — the one place non-determinism meets determinism |
| Sandboxing | execution isolation — the agent runs code! |
| Permissions | which dirs, which commands, network access |
| Recovery | checkpointing (git branches/stashes as state snapshots) |

**Verification is the keystone**: coding has objective signals (tests compile, pass, lint) that chat lacks — which is why coding agents converged on autonomy faster than other domains. The lesson generalizes: *agents go autonomous exactly as far as their verification signal reaches.*

## The risks (production-adjacent code, machine-written)

- **Plausible-wrong code**: passing tests ≠ correct — coverage gaps become agent loopholes; it optimizes the metric (Goodhart, again)
- **Supply chain via agent**: an agent that installs dependencies is an automated CVE buyer — lockfiles, allowlists, review
- **Prompt injection through the repo**: READMEs, comments, dependency docs steer agents ("helpful" instructions planted in a file) — untrusted-content law, applied to *source code itself*
- **Permission creep**: "just give it shell" — sandbox, scope, approve destructive ops
- **Review atrophy**: rubber-stamping PRs because the agent's output looks tidy — the Code Review page's discipline is *more* necessary, not less

## The DevOps mapping (this page is about *you*)

| Coding agents | Your world |
|---|---|
| Tests as evals | CI as the ground truth — you own the verification signal |
| Sandbox/permissions | container isolation, least privilege |
| Injection via repo | untrusted input — now in source form |
| Agent-written IaC/pipelines | treat as PRs from a fast junior: review, policy gates, admission control |
| Your leverage | the engineer who owns CI/evals/gates determines how autonomous agents can safely be |

## Remember This

1. Evolution: autocomplete → chat → assistant → repo-aware → agent; each stage fixed its parent's context/iteration ceiling
2. A coding agent = agent + harness with repo tools; verification via tests is the keystone
3. Autonomy scales with verification signal — why coding led
4. Risks: plausible-wrong, supply chain, injection-via-repo, permission creep, review atrophy
5. Owning the verification infrastructure = owning how autonomous agents can get

## One Sentence

Coding agents are repo-aware agents in a harness whose autonomy is bounded by the verification signal of tests and review — evolving from autocomplete by repeatedly outgrowing each stage's context and iteration limits, with all the security and review discipline that machine-authored production code demands.

## Knowledge Check

1. Why did coding agents reach autonomy before legal or medical agents?
2. Your team lifts coding-agent PRs. Which three gates from the main track apply unchanged?
3. Design the sandbox + permission set for an agent allowed to modify CI pipelines.

---

**← Previous:** [MCP](mcp.md)
**Next:** [AI Architecture](ai-architecture.md) →
