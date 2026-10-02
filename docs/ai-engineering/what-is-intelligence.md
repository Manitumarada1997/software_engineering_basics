# What Is Intelligence?

## What is this about?

Before "artificial" intelligence, we must be honest about the original word. This page builds working definitions of intelligence, information, knowledge, and learning — simple enough to use, sharp enough to build a course on.

## The four words, defined simply

- **Information**: raw facts. "Server CPU is at 95%." A log line is information.
- **Knowledge**: information compressed into *patterns that predict*. "CPU 95% during deploys is normal; at 3 AM it is not." Knowledge is information with relationships attached.
- **Reasoning**: combining knowledge to reach conclusions you were never directly told. "CPU is high, no deploy, at 3 AM → probably a runaway job."
- **Learning**: acquiring knowledge from experience *without being explicitly given the conclusion*.

**Pattern recognition** is the engine underneath learning: noticing that things which looked different in detail behave the same in structure.

## Intelligence, one working definition

Not a philosophical treatise — an engineer's definition:

> **Intelligence is the ability to use experience to predict and act in new situations.**

Notice what this does *not* require: consciousness, understanding "in the human sense", or having a soul. It requires: past experience → internal adjustment → better future behavior. By this definition, a spam filter is a sliver of intelligence, and a mouse is more of one.

## The four-way comparison (the table this page exists for)

| | Traditional software | Machine learning | Human intelligence |
|---|---|---|---|
| Knowledge source | human writes rules | extracted from data | experience + culture + teaching |
| Handles new situations | only as foreseen by the author | only as represented in training data | generalizes, sometimes wildly |
| Explains itself | yes (read the code) | partially (the rules are numbers) | partially (we confabulate!) |
| Makes mistakes | only when rules are wrong | confidently, in unexamined ways | constantly, creatively |
| "Intelligence" | none — perfect execution | narrow prediction | broad, general |

**Artificial intelligence** = any system that performs tasks which, in humans, would require intelligence — *by any mechanism*. This includes rule-based systems (they were called AI in 1970!), statistics, and neural networks. The definition is about the task, not the mechanism.

## What makes something "intelligent"? (the honest test)

The useful test is **generalization to novelty**: does the system behave sensibly in situations it was never explicitly prepared for?

- A thermostat: no (nothing new ever happens to it)
- A rule-based chatbot: no (edge cases break it — the previous page's wall)
- A spam filter: slightly (new spam variants, sometimes caught)
- An LLM: substantially (novel questions, novel phrasings) — *this* is why the 2022 moment felt different
- A human: maximally (and sometimes disastrously — generalizing from small samples is also the root of prejudice)

## Automation vs intelligence — the distinction to never blur

**Automation** executes known steps faster; **intelligence** decides *which* steps. The history of computing is automation; the ML turn is delegating the *deciding*. When a system both decides and acts (an agent on your cluster), you've combined both — and inherited the risks of each.

## What problem did this create?

Once we committed to "intelligence = learned prediction," we needed machinery for learning itself: What is being adjusted? Adjusted toward what goal? That machinery — models, weights, loss — is the next three pages.

## Remember This

1. Information → knowledge (patterns) → reasoning (combining) → learning (acquiring)
2. Intelligence (working definition): experience → adjustment → better prediction on new situations
3. AI = tasks, not mechanisms — rules were "AI" in 1970
4. The real test: generalization to novelty; that's why LLMs felt different
5. Automation executes known steps; intelligence chooses steps — agents do both

## One Sentence

Intelligence is the capacity to convert experience into predictions about new situations, and artificial intelligence is any mechanism that achieves human-like tasks — a definition about tasks and generalization, not about inner experience.

## Knowledge Check

1. Place on the automation–intelligence spectrum: cron job, spam filter, junior engineer, LLM agent.
2. Why is "generalization to novelty" a better test than "acts human"?
3. Your CI pipeline is automation. Where could a *learned* component legitimately help?

---

**← Previous:** [Track Overview](index.md)
**Next:** [From Calculators to Learning Machines](from-calculators-to-learning-machines.md) →
