---
title: "AI SDLC: An Opinionated Blog for the World's Largest ATM Company"
description: 'A five section AI SDLC pipeline from brown field Angular to React migration work: rules before models, scope fixed before code, and people at the gates, not in the middle.'
pubDate: '2026-10-07'
heroImage: '../../assets/blog/ai-sdlc-for-the-worlds-largest-atm-company/cover.png'
---

## Why I am writing this

Most AI SDLC writing starts from a blank page. Real companies do not have a blank page.

Over the past months I worked on a set of feasibility projects for the World's Largest ATM Company. I will call them ATMCo from here on. Some projects were green field. Most were brown field: old Angular screens that had to become React, inside software that is certified and runs on machines that hold cash. To give you a sense of size, one customer journey alone had 52 components and 143 routes.

Out of that work came a way of building software with AI that I am willing to defend. It is opinionated. It respects how ATMCo is built today, and it also says clearly what has to change for the company to be ready for AI.

The brown field migration pipeline is the spine of this blog, because that is where the hard lessons were. I will point out what carries over to green field as we go.

One honest note before we start. This is a design that came out of feasibility work. Parts of it are proven, parts are still open, and I will tell you which is which.

[![The five section AI SDLC pipeline, with the shared bus on top and the Pattern Library under it](/images/blog/ai-sdlc-pipeline.png)](/images/blog/ai-sdlc-pipeline.png)

*Click the diagram to enlarge it.*

Read it left to right. Five sections: Requirements, Context, Orchestration and Code Gen, Validation, and PR and Monitoring. Green boxes are AI steps. Yellow boxes are people. The pink box around Section 5 is future state, not something running today, and so is the blue visual diff box in Section 4 (4.2).

Two things sit above the five sections and matter more than any single box:

- **The Event, State and Token Budget Bus** (top bar). Every step publishes what it did and what it spent. It also remembers which routes are already migrated and which tests already exist, so the next run reuses them or only builds the difference.
- **The Pattern Library for Deterministic Fixes** (blue shape). A growing set of code rewrite rules that fix known problems with no model call at all.

## Three opinions that shape everything

Before the walk through, here is what I believe. If you disagree with these, you will disagree with the diagram.

1. **Use the model last, not first.** If a rule can do the job, the rule does it. The model is called only for the part that needs judgment: business logic, RxJS conversions, the shape of the new code.
2. **Fix the boundary before you write any code.** A model given a loose brief will widen the job. So scope and context are agreed and signed off as files before a single line is generated.
3. **People sit at the gates, not in the middle.** There are four human sign offs on the diagram (1.2, 2.4, 4.3 and 5.2). Each one reviews a small, readable file. Nobody is asked to babysit a model while it types.

## Section 1: Requirements. Draw the fence first

Look at the two boxes on the far left of the diagram.

**1.1 Input Data Ingestion** takes the raw material: the Jira ticket, the acceptance criteria and the target route paths. A small model turns that into one file, `route_spec.json`. It also runs a plain check that the route really exists in the code base.

**1.2 Scope Approval** is a person. The Engineering Lead reads `route_spec.json` and publishes a scope approved event to the bus. Nothing moves until that event is there.

Why start here? Think about it. The most expensive mistake in this pipeline is doing good work on the wrong thing. A wrong route or a stale ticket burns tokens through four more sections before anyone notices.

The sign off also covers where the input came from. On the diagram this is the note under 1.2: an automated check or a person confirms the source of the spec before it goes in. In a regulated business that record is not optional.

**What carries to green field:** all of it. Replace the route path with a feature slice and the idea holds.

## Section 2: Context. Give the model less, not more

The second column has four boxes, and this is where most of the quality is won or lost.

**2.1 Angular AST Context.** We do not hand the model raw source files. A parser turns the Angular code into an Abstract Syntax Tree, which is the code as a tree of parts instead of plain text. We strip comments, CSS and formatting, then pull out what matters: component inputs, methods, dependencies, state. A reasoning model writes this up as `tech_spec.md`, with API contracts, state models and the component hierarchy.

**2.2 Design System Alignment.** A small model maps each legacy UI element to an approved React design system component and writes `component_map.json`. Accessibility checks are logged here too.

**2.3 Technical Specifications.** Functional and non functional needs are pulled together into the approved spec.

**2.4 Context and Architecture Sign off.** A developer or architect reads the spec and the component map and approves them.

The way to think about it: the model should never have to guess. If it does not know which button component to use, it will invent one. If it does not know the approved state library, it will pick a popular one. Section 2 removes those guesses before code gen starts, and leaves a paper trail that maps each old route to its new spec.

There is one open gap, and it is marked in pink on the diagram. `component_map.json` links old Angular to new React today. What happens when a production bug shows up six months later? That map has to live on as a maintained record, or the trace is lost. We have not solved this yet.

**What carries to green field:** 2.2, 2.3 and 2.4. There is no legacy tree to parse, but the design system binding and the signed spec matter just as much.

## Section 3: Orchestration and code gen. One planner, six narrow workers

This is the tall middle column, with the purple diamond at the top.

**3.1 Central Planner** (the diamond). A reasoning model reads the approved spec and writes `plan_manifest.json`. That file is a task tree with clear, non overlapping bounds for each worker, and a token budget for each task. The budget is audited, which is the red note inside the diamond.

**3.2 Parallel workers.** Six workers run side by side, each in its own small context window:

| Box | Worker | What it produces |
| --- | --- | --- |
| 3.2a | Component Migration | React presentation components |
| 3.2b | Controller Migration | Controller logic moved into React hooks |
| 3.2c | Template Migration | Angular `*ngIf` and `*ngFor` turned into JSX |
| 3.2d | Route Integration | Route guards and page routing on TanStack Router |
| 3.2d (second box) | Service Migration | Angular services turned into typed TypeScript services |
| 3.2e | Unit Test | Test files written from the interface spec only |

Notice the last row. The test worker sees the interface, not the generated code. An agent that writes code and then writes its own tests tends to write tests that agree with its own mistakes. The purple note at the bottom of the diagram goes one step further and proposes a separate critic whose only job is to break the generated React.

**The red arrows.** Each worker runs its own lint and compile loop. When something fails, it does not call the model again straight away. It checks the Pattern Library first. If a known rule matches, a codemod fixes it for zero tokens. Bulk import renames and standard syntax conversions never reach a model at all.

**3.3 Parity Check and Aggregator.** A rule engine plus a small model joins the six outputs and compares the result against the plan and the approved spec. Hooks, state files and components must match the contract. The blue line on the left of the diagram is the feedback loop back to the workers when they do not. Code is committed only when parity passes.

The payoff is simple. Six workers in a row take the sum of their times. Six workers side by side take the time of the slowest one, and each carries a much smaller context.

**What carries to green field:** the planner, the token budgets and the split between who writes code and who writes tests. The codemod layer matters less when there is no legacy code to rewrite.

## Section 4: Validation. Machines check first, people check last

The fourth column has three boxes. Two of them, 4.1 and 4.3, are the working design. The blue box in the middle, 4.2, is not built yet, so I cover it with the future state below.

**4.1 Validation Agent.** Runs the test suite in a sandbox: Jest coverage, the linter and a static security scan. The checking tools are plain, repeatable tools. The model's job is to read the results and send failures back to Section 3. Each time an agent misses something, the miss is written into a `skills.md` file so the next run starts smarter. Output: `validation_report.json` and `coverage_report.html`.

**4.3 Code and PR Sign off.** An architect or senior engineer reads the reports, then approves.

You see, the person at 4.3 is not reading every line hoping to spot a bug. They are reading evidence: tests ran, coverage is here. That is a much better use of a senior engineer's hour.

One item is still open, marked in pink on the diagram:

- **Subtask completion audit.** How do we prove that each piece of built functionality maps to an automated test case that runs? Coverage numbers do not answer that.

**What carries to green field:** 4.1 and 4.3 as they are.

## Section 5: PR and monitoring. The part that is still future state

The last column sits inside a pink border under the label Future State. I want to be straight about that. Sections 1 to 4 are the working design, apart from the visual diff box in Section 4. Section 5 is where it should go next, and that box comes first.

**4.2 Visual Regression and Pixel Diff** (the blue box in Section 4). The old Angular screen and the new React screen are rendered side by side in a headless browser with Playwright or Puppeteer. Snapshots are compared at pixel level and across screen sizes. We still need to pick and prove the visual testing tool, which is marked in pink on the diagram: this box is a discussion item, not a decision. On green field it changes shape, because there is no old screen to compare against. You compare against the design file instead.

**5.1 Autonomous PR Node.** A small model assembles the pull request, attaches the audit logs, the changelog and the validation summary, and notifies reviewers. Branch protection rules stay enforced.

**5.2 PR Review and Merge.** The diagram splits here into two boxes, and the split is the idea:

- **Left box, AI review.** Low regression risk and a high confidence score. A model checks the compliance logs and scores and approves, or fast tracks the approval.
- **Right box, human review.** High regression risk, low confidence or low PR quality. A senior engineer does a full peer review.

Review effort follows risk. A renamed import and a rewritten cash dispense flow should not wait in the same queue.

**Production monitoring.** The plan adds a monitoring step after merge that watches live errors and can trigger a rollback. It is in the written design but not yet drawn as a box on the diagram.

**The flywheel.** Follow the long line from 5.2 back up to the Pattern Library. When a reviewer rejects a PR or edits the generated code by hand, that feedback is normally lost. Here, the difference between what the AI wrote and what the person merged is captured and turned into a new rule in the library. The same mistake should not cost a human twice.

In my view this loop is the real asset. Models will change every few months. The library of fixes that your own engineers taught the pipeline stays with you.

## Is this feasible? What other engineering teams found

An opinion is cheap. So here is what three teams reported when they ran migrations with the same core idea, rules first and model second.

| Team | What they migrated | What they reported |
| --- | --- | --- |
| [Slack](https://slack.engineering/balancing-old-tricks-with-new-feats-ai-powered-conversion-from-enzyme-to-react-testing-library-at-slack/) | Over 15,000 Enzyme tests to React Testing Library | About 80% converted automatically. Feeding AST conversions into the model [beat the model alone by 20 to 30%](https://www.infoq.com/news/2024/12/ai-enzyme-react-test-library) |
| [Airbnb](https://www.infoq.com/news/2025/03/airbnb-llm-test-migration/) | Nearly 3,500 React test files | 75% done in four hours, 97% after four more days of tuning, fewer than 100 files fixed by hand. A 1.5 year estimate became six weeks |
| [Google](https://arxiv.org/pdf/2504.09691v1) | 39 migrations, 595 code changes | 74% of changes written by the model. Developers estimated half the time of doing it by hand |

Three lessons from these teams line up with the diagram:

1. **The hybrid wins.** Slack tried AST only and model only before joining them. That is the red arrow path in Section 3.
2. **The last stretch is the expensive part.** Airbnb got to 75% in an afternoon and then worked for days on the rest. Plan for a long tail. Do not promise 100%.
3. **Reviewers become the limit.** In a [second Google report](https://arxiv.org/html/2501.06972v1), the team capped how many changes went out each week so reviewers were not flooded. That is the case for the risk split at 5.2.

One caution so I do not oversell. All three examples are narrower than ours. They moved tests or made type changes. Moving a whole Angular screen to React, with state and routing, is a harder job. Expect the automatic share to be lower than these numbers until your own pattern library fills up.

## Where I would still argue with my own diagram

An opinionated blog should also say where the opinion is weak. Four places.

1. **The split inside one route.** Airbnb ran files side by side, and files are independent. Our six workers split one route by concern: component, controller, template. Those parts depend on each other. The parity check at 3.3 is where that dependency gets paid for. If 3.3 keeps bouncing work back, the answer is fewer, wider workers, not more retries.
2. **100% coverage as a gate.** Box 4.1 asks for 100% Jest coverage. Coverage tells you lines ran, not that behaviour is right. I would rather gate on the open question in pink: does every acceptance criterion map to a test that runs?
3. **Pixel diff on a new design system.** If the new React screens use new design tokens, they will differ from the old screens on purpose. A strict pixel diff will then flag everything. It needs a tolerance, or a check on layout and behaviour instead of pixels.
4. **A diff is not a rule.** The flywheel says human edits become new codemods automatically. One human edit is one example. Turning it into a safe, general rule needs a person to review it before it enters the library. Otherwise one reviewer's habit becomes everyone's default.

## What ATMCo keeps, and what it has to change

This design leans on how ATMCo already works. That was deliberate.

**What stays.** Jira tickets and acceptance criteria are still the input. Engineering Leads, architects and senior engineers still sign off. Jest, the linter, the security scan and branch protection are the same tools the teams trust today. The pipeline runs as a workflow inside the software studio and IDE plugin the teams already use, so nobody learns a new place to work.

**What has to change.** Four things, and none of them are about models.

1. **Specs become real files.** `route_spec.json` and `tech_spec.md` are reviewed, versioned and owned like code. A ticket with two lines of text cannot feed this pipeline.
2. **The design system becomes machine readable.** Box 2.2 only works if every approved component is listed in a form a model can look up.
3. **Someone owns the Pattern Library.** It needs an owner, a review step and a release cycle, the same as any shared library.
4. **Tokens get a budget line.** The bus tracks spend per task. Teams should see that number the way they see build minutes today.

Think about it this way. The model is the easy part to buy. The four items above are what make a company ready for it, and they take longer.

## The whole thing on one table

| Section | Job | Files it produces | Why it matters |
| --- | --- | --- | --- |
| 1. Requirements | Fix the scope | `route_spec.json` | Stops the job from growing |
| 2. Context | Parse the old code, bind the design system | `tech_spec.md`, `component_map.json` | Smaller prompts, no invented UI |
| 3. Orchestration | Plan, generate in parallel, check parity | `plan_manifest.json`, merged bundle | Speed, plus zero token fixes |
| 4. Validation | Test, scan | `validation_report.json`, `coverage_report.html` | People review evidence, not raw code |
| 5. PR and Monitoring | Route review by risk, learn from edits | Pull request, new library rules | The pipeline gets better with every merge |

## A test you can run on your own pipeline

Pick the last ten fixes a human made to AI generated code on your team. Ask one question of each: will the pipeline make that same mistake again next week?

If the answer is yes for most of them, you do not have an AI SDLC yet. You have a fast typist. Start with the Pattern Library and the line that feeds it.

Let's do it.
