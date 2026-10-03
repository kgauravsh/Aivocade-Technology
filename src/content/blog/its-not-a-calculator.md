---
title: "It's NOT a Calculator"
description: 'My son checked his maths board exam answers with Gemini. Why an LLM predicts the answer instead of computing it, and how modern AI gets maths right.'
pubDate: '2026-03-09'
heroImage: '../../assets/blog-placeholder-about.jpg'
---

This happened today - My son who gave Maths boards exam today, was looking at gemini to validate his answers ...

So I checked with him - What does he consider the machine he is using (In this case Gemini). Very usual answer - Its computing - Its smart and what not !!

\>>>> I had to correct him.

## The Core Mechanism to be understood is Pattern Matching, Not Computation

An LLM does not compute math. It predicts the most probable next token based on patterns learned from training data.

When you ask "what is 847 × 23?", the model is essentially doing:

> "I've seen thousands of multiplication examples. The token sequence that statistically follows this pattern is... 19481"

It's doing linguistic pattern completion, not arithmetic.

**Fun fact** - GPT-4 gets ~94% on simple arithmetic but drops significantly on multi-step or large-number problems - not because it's "dumb", but because it was never designed to compute.

## What happens behind the scene (for developers to understand)

Modern AI Do Math Reliably because it has figured out the following way:

**Tool Use / Function Calling (The Right Pattern)**

```text
LLM -> recognizes math intent
    -> calls Python interpreter / calculator tool
    -> gets deterministic result
    -> weaves answer into response
```

This is what Claude, GPT 4, and Gemini do when they "solve math" correctly.

**The LLM's job is intent recognition + orchestration, not computation.**
