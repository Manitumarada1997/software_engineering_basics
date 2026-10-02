# Neural Networks

## What is it?

A **neural network** is many simple units ("neurons") stacked in layers, where each neuron is exactly the house-pricer from two pages ago: multiply inputs by weights, add up, output a number. Stacked and interconnected, they can approximate almost any pattern — not because any one neuron is clever, but because **composition of simple bends creates arbitrary complexity**.

## Why does it exist?

A straight line (`price = w x size + b`) cannot separate "cat photo" from "not cat" — real concepts are not straight lines. Two options: hand-craft thousands of features (the old way, exhausted by the 1980s) or **let layers of simple units bend the space until the pattern becomes separable**. Neural networks are the second option, loosely inspired by brain neurons (the inspiration is historical; the math is what matters).

## The neuron, completely (you already know it)

    input1 x w1 + input2 x w2 + ... + bias  ->  activation function  ->  output

The **activation function** is a small non-linear bend (e.g., "if negative, output 0"). Without it, a million stacked neurons collapse into one big linear formula — the bend is what makes depth *mean* something. A network is thousands of these units wired in layers:

    inputs -> [layer 1] -> [layer 2] -> ... -> [layer N] -> output

**Hidden layers** = the middle layers, where intermediate features form: early layers learn edges (in vision) or letter pairs (in language); deeper layers learn shapes, objects, phrases — *features nobody labeled*. This self-building of intermediate concepts is why neural networks replaced hand-crafted features.

## Forward pass and backpropagation — the two verbs

- **Forward pass**: input flows through the layers to a prediction. This is all that happens at *inference* time.
- **Backpropagation**: during training, the loss (wrongness) at the output is sent *backward* through the same network, and each weight receives its share of the blame — its gradient. Then gradient descent nudges it. Forward to guess, backward to blame, nudge, repeat.

Backpropagation (popularized 1986, Rumelhart–Hinton–Williams) answers why 1958's one-layer perceptron stalled: nobody could train multiple layers. With blame assignment solved, depth became trainable — and the 2012 explosion (next page) became possible.

## Why networks generalize (and when they don't)

A network with enough neurons can memorize anything (overfitting — previous page). What makes networks valuable is that training on *lots of varied data* pushes them toward *the simplest pattern that fits*. Data volume is the regularizer: with ten examples, memorize; with ten million, learn the actual shape.

## The DevOps mapping

| Neural net | Your world |
|---|---|
| Forward pass | request through a pipeline of stages |
| Training loop | a feedback reconciler with the loss as error signal |
| Backprop | blame assignment — tracing an incident backward through the causal chain |
| Activation | the non-linearity that keeps stages from collapsing into one formula |

## What problem did this create?

Networks could now be trained — but training big ones needed two missing ingredients: **data at scale** and **compute at scale**. How both arrived — and what deep learning then conquered and could not — is the next page.

## Remember This

1. A neuron = weighted sum + a small non-linear bend; a network = layers of them
2. Composition of simple bends = arbitrarily complex patterns — depth's whole trick
3. Hidden layers self-build intermediate features nobody labeled
4. Forward = guess; backprop = assign each weight its blame; gradient descent = nudge
5. Lots of varied data pushes toward generalization; little data invites memorization

## One Sentence

A neural network is a stack of tiny weighted-sum-plus-bend units whose trained depths build unlabeled intermediate features, with backpropagation assigning each weight its share of the error so gradient descent can improve it.

## Knowledge Check

1. Remove all activation functions: what happens to a 50-layer network, mathematically?
2. Explain backpropagation using "incident blame assignment".
3. Why do hidden-layer features matter more than the labels for hard tasks?
4. Why does more data push toward general patterns rather than memorization?

---

**← Previous:** [What Is Learning?](what-is-learning.md)
**Next:** [The Deep Learning Era](deep-learning-era.md) →
