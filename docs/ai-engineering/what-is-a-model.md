# What Is a Model?

## What is it?

A **model** is a mathematical machine: it takes an input, multiplies it by internal numbers, and produces an output. The internal numbers are called **parameters** or **weights**. That's the whole secret — everything else is scale.

!!! info "Parameter — one sentence"
    A parameter is an adjustable number inside the model that controls how input becomes output.

## Why does it exist?

The previous pages established: some knowledge can't be written as rules. A model is the *container* for that unwritable knowledge. Instead of instructions, it stores **numbers tuned to fit examples**. The knowledge lives in the tuning.

## The simplest model in existence (follow this arithmetic)

Predict a house price from its size. A model with **one parameter**:

    price = weight x size            (weight = 300)
    size  = 100 m2   ->  price = 300 x 100 = 30,000

The weight *is* the model's entire knowledge ("each m2 adds 300"). Better model, add a second parameter:

    price = weight x size + bias     (weight = 320, bias = -5000)

**Bias** is another adjustable number — the model's guess when all inputs are zero. Two parameters: a *line*. A small neural network: millions of parameters. GPT-scale models: hundreds of billions:

    house pricer:      2 parameters       (a line)
    spam filter:       ~10,000            (a curve through word patterns)
    LLM:               ~700,000,000,000   (a surface through language itself)

**When you read "a 70B model"** — that's 70 billion adjustable numbers. Nothing more mystical than `weight x size + bias`, multiplied by scale.

## Model vs program — the comparison that makes everything click

| | Program | Model |
|---|---|---|
| Content | instructions | numbers (weights) |
| Written by | humans | adjusted by training (next pages) |
| Runs how? | line by line | input multiplied through the weights to output |
| File looks like | source code | a giant array of decimals |
| Debugging | read the code | you cannot read a billion numbers — hence evals |

The file you download when you run a local model later (a .gguf file) **is** this: billions of weights, as numbers on disk. A model file is an artifact — your CI/CD instincts about artifacts, versioning, and immutability apply exactly.

## Weights, together, are behavior

Change one weight in the house pricer from 300 to 310: different predictions everywhere. Multiply that sensitivity by 70 billion: models hold enormous, interconnected patterns — French verbs, Python syntax, the tone of a corporate email — each smeared across millions of weights, none individually meaningful.

This is why you cannot edit one fact into a model (a recurring question!): facts are not stored at addresses; they are ripples across the whole surface. Fixing knowledge means retraining or retrieval — a theme this track returns to.

## What problem did this create?

We have a machine with adjustable numbers. Who adjusts them? Toward what target? The answer — the adjustment procedure called **learning** — is the next page.

## Remember This

1. Model = input multiplied by weights to output; parameters are the adjustable numbers
2. A 70B model is 70 billion numbers on disk — an artifact, versionable like any artifact
3. Two parameters make a line; billions make a surface through language
4. Knowledge is smeared across weights — no single fact lives at a single address
5. Programs you read; models you cannot — the birth of evals over debugging

## One Sentence

A model is a mathematical function whose behavior is stored in adjustable numbers called weights, and everything from a two-parameter line to a 70-billion-parameter LLM is the same idea at different scale.

## Knowledge Check

1. In `price = 320 x size - 5000`, identify the parameters and what each contributes.
2. Why cannot you delete one wrong fact from a large model's weights?
3. Why do your artifact-management instincts transfer to model files?
4. Explain "70B model" to a colleague without using the word "neural".

---

**← Previous:** [Probability & Prediction](probability-and-prediction.md)
**Next:** [What Is Learning?](what-is-learning.md) →
