---
title: 'How tokens maths work with an example'
description: 'A real agentic bug fix, turn by turn: why tokens explode through history and retries, and how better reasoning and compaction cut them.'
pubDate: '2026-06-15'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

## Real agentic coding scenario

### The Scenario: Fix a Bug in a Large Codebase

The dark factory pipeline gets a JIRA ticket that says (a defect maybe)

> "Payment service throws NullPointerException when user has no saved address during checkout"

### What "Input Tokens" Means (What Your Client Is Thinking)

Input tokens = what you send to the model in each request:

- System prompt
- The task description
- Code context you load

### Where Tokens Actually Explode in Agentic Loops

A bug fix in an SDLC agent isn't one request. It's many turns. Here's what actually happens:

**Turn 1 { Understanding }**

```text
[System prompt: 2,000 tokens]
[Task: 200 tokens]
[Loaded files: PaymentService.java, CheckoutController.java = 3,000 tokens]
-> Model responds: "I need to see AddressRepository.java too"
Total: ~5,200 tokens
```

**Turn 2 { Investigation }**

```text
[Everything from Turn 1: 5,200 tokens]  ← history carried forward
[AddressRepository.java: 1,500 tokens]
-> Model responds: "Found the issue. Line 47. Let me check the test suite"
Total: ~7,000 tokens
```

**Turn 3 { Fix Attempt 1 }**

```text
[Everything from Turns 1+2: 7,000 tokens]
[PaymentServiceTest.java: 2,000 tokens]
-> Model writes a fix. Tests fail.
Total: ~10,000 tokens
```

**Turn 4 { Retry }**

```text
[Everything from Turns 1+2+3: 10,000 tokens]
[Test failure output: 800 tokens]
-> Model tries again. Wrong approach again.
Total: ~11,500 tokens
```

**Turn 5 { Retry again }**

```text
[Everything above: 11,500 tokens]
-> Correct fix finally produced
Total: ~13,000 tokens
```

**Grand total for one bug fix: approx 46,700 tokens across all turns.** The system prompt (your "input") was only 2,000 tokens. The explosion happened in history accumulation and retries.

## Probable Solution

Different models tackle these problems differently. As an example, What GPT-5.5 / Codex Actually Fixes

### Problem 1 — Retry loops (Output token waste)

GPT-5.4 might take 3-4 attempts to nail the fix. GPT-5.5's better reasoning gets it in 1-2. That alone cuts 30-40% of total tokens consumed - not because input shrank, but because the loop terminated sooner.

### Problem 2 — History bloat (Compaction)

Without compaction (GPT-5.4 default), Turn 5 (Retry Again) carries the full verbatim transcript of every prior turn. With compaction (\*), Codex summarizes older turns into a compact memory:

Instead of carrying:

```text
[Turn 1 full text: 5,200 tokens]
[Turn 2 full text: 1,800 tokens]
```

It carries:

```text
[Compacted summary: "Investigated PaymentService.
 Root cause: null check missing at line 47. Address Repository confirmed. Tests located."
 = 80 tokens]
```

A 70-80% reduction in history tokens, which is the dominant cost in any multi-turn agent session.

**Note.**

\* GPT-5.1-Codex-Max was the first model natively trained to operate across multiple context windows through a process called compaction.
