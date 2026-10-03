---
title: 'I Tested a Harness on a Coding Agent. It Fixed Exactly One Thing.'
description: 'Same model, same task, same repo, one rules file added. The baseline missed the same unstated convention in 5/5 runs; the harness run fixed just that, and nothing else moved.'
pubDate: '2026-10-03'
heroImage: '../../assets/blog/testing-a-harness-on-a-coding-agent/harness-image.jpeg'
---

This is round two of a small side project I'm calling Harness Lab: does handing a coding agent
an explicit harness :  a written set of house rules :  actually change what it gets right, or does
it just feel like it should? Round one ran on a small library app (overdue fines, borrowing
limits) and already showed the same pattern: a baseline agent missing one unstated convention,
every single run. For round two I wanted a different domain, so I weighed a few options : 
subscription billing with proration, appointments across time zones, a multi-tenant task tracker,
concurrent-order inventory :  and picked the task tracker, because a tenant-isolation bug is the
scariest kind to watch happen live: not a wrong number, but one customer's data leaking into
another's.

## The question

Does writing down your house rules for a coding agent actually change anything?
So I ran a small controlled experiment: same model, same task, same repo, five runs with nothing
but the task prompt, then five more runs with exactly one rules file added to the system prompt.
Score every run the same way, automatically, on checks the agent never sees.

## The setup

The task: a multi-tenant task tracker :  tenants → projects → tasks. I asked the agent to add three features to an existing codebase:

1. Bulk-reassign every open task in a project from one person to another, with a cap on how many open tasks anyone can hold.
2. Archive a completed task.
3. Compute the "lead time" (in whole days) between a task's creation and its completion.

Every one of those three functions takes a `tenant_id` argument. We never say why.

Underneath, the grader checks 8 things, 5 of them mechanical (tests pass, tests were written, no
`print()`, no naive datetimes, the API layer doesn't reach past the service layer into storage),
and 3 of them real house rules that live nowhere in the code or the task text:

- A `tenant_id` has to actually be *checked*, not just accepted :  reassigning or archiving across
  a tenant boundary should be rejected, not silently allowed.
- "Whole days" means calendar days in the Asia/Kolkata timezone, not a raw 24-hour span.
- The open-task cap is counted across *every* project someone has in that tenant, not just the
  one project being touched.

The agent never sees these three. It only finds out it got one wrong when we tell it.

## Run it five times with nothing

Five runs, no harness, nothing but the task prompt. Every single one landed on **7/8** : 
and failed the *same* check, every time:

| Run | Score | Cost | Turns | Failed check |
|---|---|---|---|---|
| 1 | 7/8 | $0.26 | 15 | lead-time days |
| 2 | 7/8 | $0.26 | 16 | lead-time days |
| 3 | 7/8 | $0.26 | 21 | lead-time days |
| 4 | 7/8 | $0.22 | 10 | lead-time days |
| 5 | 7/8 | $0.37 | 19 | lead-time days |

Same check, same error, every run:

```text
AssertionError: 3 != 4
```

The agent computed lead time with a plain `(end - start).days` :  a raw elapsed-time floor. The
correct answer required converting both timestamps to Asia/Kolkata first, *then* subtracting
dates. Off by exactly one day, every time, because the subtraction crossed a local midnight the
UTC math never saw.

The two *other* unstated rules :  tenant scoping and the cross-project cap :  it got right in all
5 runs, unprompted. And every single run reported itself "done." It never once suspected the one
thing it got wrong.

## Add one file, run it again

I wrote a short rules file :  the kind of thing a team would normally put in a `CLAUDE.md` : 
stating the conventions in plain prose, not the test's actual numbers:

> Whenever a rule talks about "days," it means calendar days in Asia/Kolkata, not a raw 24-hour
> span. Convert to Asia/Kolkata before taking the date, then subtract dates :  never subtract
> UTC-aware datetimes directly for a "days" business rule.

Same task. Same repo. Same model. Appended that one file to the system prompt. Ran it five more
times:

| Run | Score | Cost | Turns | Failed check |
|---|---|---|---|---|
| 1 | 8/8 | $0.27 | 15 | :  |
| 2 | 8/8 | $0.25 | 14 | :  |
| 3 | 8/8 | $0.25 | 14 | :  |
| 4 | 8/8 | $0.25 | 15 | :  |
| 5 | 8/8 | $0.24 | 13 | :  |

5/5, no failures.

## What actually moved

This is the part that matters more than the clean 8/8 :  *what didn't change*:

| Check | No harness | With harness |
|---|---|---|
| its own tests pass | 5/5 | 5/5 |
| it wrote tests at all | 5/5 | 5/5 |
| tenant scoping checked | 5/5 | 5/5 |
| **IST calendar days, not raw 24h** | **0/5** | **5/5** |
| cross-project overload cap | 5/5 | 5/5 |
| layer boundary respected | 5/5 | 5/5 |
| log events registered | 5/5 | 5/5 |
| no print / naive datetimes | 5/5 | 5/5 |

One row changed. Nothing else moved in either direction :  the checks that were already reliable
stayed reliable, and the harness didn't accidentally break something else while fixing this.
Cost and turn count also dipped slightly with the harness in place (mean $0.254 vs $0.277, 14.2
turns vs 16.2) :  plausibly because the agent spent fewer turns re-deriving a convention it no
longer had to guess at, though that's a secondary effect next to the correctness flip.

## Why this is the useful kind of evidence

It's easy to find one example of an agent getting something wrong and conclude "it needs more
guardrails." It's much more useful to show the *exact same mistake, every time, with nothing
else varying* :  and then show that adding exactly one stated rule removes exactly that mistake
and nothing more.

That's the actual case for a harness: not that an agent is unreliable in general, but that it is
*reliably* wrong about the specific category of thing nobody told it :  team conventions that
aren't inferable from the code, the kind every codebase accumulates and never writes down. A
harness isn't making the model smarter. It's handing over the one fact it was missing.
