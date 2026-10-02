# What Is Learning?

## What is it?

**Learning** is the procedure that finds good values for a model's weights. Given examples and a measure of wrongness, it repeatedly adjusts weights to be *less wrong*. Three words carry the whole mechanism:

- **Loss** — a number measuring how wrong the model currently is
- **Gradient** — the direction each weight should move to reduce that wrongness
- **Gradient descent** — the loop: measure wrongness, nudge every weight downhill, repeat

## Why does it exist?

A model with random weights is useless (random predictions). A human can't set 70 billion numbers by hand. Learning is the automatic adjustment procedure — the answer to "who adjusts the weights?"

## The entire mechanism, with one weight (follow the arithmetic)

House pricer `price = w x size`, examples from real sales: 100 m2 sold for 35,000.

    Try w = 300:  prediction 30,000. Wrongness = |35,000 - 30,000| = 5,000   ← the LOSS
    Try w = 350:  prediction 35,000. Wrongness = 0                             ← perfect on this example

How would you find 350 without guessing? Try w, see the error, and adjust in the direction that reduces it:

    w too low (predicted under actual)  ->  increase w
    w too high                          ->  decrease w
    size of the nudge proportional to the error

That rule — *nudge each weight opposite its contribution to the error* — is gradient descent. "Gradient" = the slope of the wrongness; "descent" = walk downhill. One weight: obvious. A billion weights: the same rule, computed for all of them at once by a beautiful algorithm (backpropagation, next page) — because the loss is just arithmetic, its slopes are computable.

## Training — the word, defined

!!! info "Training — one sentence"
    Training = feeding examples through the model, computing loss, and running gradient descent, millions of times, until predictions fit.

The output of training is not code — it is the final weight values, frozen into a file. (Inference — *using* those weights — is the counterpart, and gets its own page.)

## Overfitting — the trap that explains half of AI's failures

A model can reduce training loss by *memorizing* the examples instead of learning the pattern:

    Training: "100 m2 -> 35,000" memorized exactly. Loss = 0.
    New house: 105 m2 -> nonsense, because it never saw 105.

**Overfitting** = fitting the noise/memorizing the examples. **Underfitting** = too simple to catch even the pattern. The profession's answer — the same discipline as test/train separation everywhere in engineering:

    Training data    (adjust weights on this)
    Validation data  (check generalization during training — tune choices)
    Test data        (final exam, touched once, never trained on)

A model that aces training data and fails test data has memorized; a deployment that aces staging and fails production has a different test/train gap — you have lived this law in another costume.

## The DevOps mapping (your existing intuition, repurposed)

| ML term | Your equivalent |
|---|---|
| Loss | a failing test's severity score |
| Gradient descent | a feedback loop (reconciler!) converging desired toward actual |
| Training run | a pipeline job producing an artifact |
| Validation/test split | staging vs production |
| Overfitting | "works in staging" — teaching to the test |
| Convergence | steady-state in a control loop |

Learning is a control system: error signal, corrective adjustment, convergence — vocabulary that should feel like home.

## What problem did this create?

One weight is a line; lines can't represent "catness" or grammar. We need models that can bend into *any* shape — stacks of simple units composing into arbitrarily complex functions. That is the neural network — next page.

## Remember This

1. Learning = loss + gradient descent: measure wrongness, nudge every weight downhill, repeat
2. Training output is frozen weights (an artifact), not code
3. Overfitting = memorizing examples; the cure is held-out test data — the staging/prod law
4. It's a control loop: error signal, adjustment, convergence — your existing mental model
5. Backpropagation (next page) is just "compute each weight's share of the error, at scale"

## One Sentence

Learning is the automatic procedure of nudging a model's weights downhill on its own wrongness — a feedback loop that converts examples into stored patterns without anyone writing rules.

## Knowledge Check

1. Define loss, gradient, and gradient descent in one sentence each, using "downhill".
2. Your model scores 99% on training data, 61% on test data. Diagnosis and cure?
3. Map training/validation/test onto your CI/staging/prod discipline. What's different?
4. Why does "the loss is just arithmetic" make automatic adjustment possible at all?

---

**← Previous:** [What Is a Model?](what-is-a-model.md)
**Next:** [Neural Networks](neural-networks.md) →
