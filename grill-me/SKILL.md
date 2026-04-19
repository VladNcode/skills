---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

## Goal

Stress-test the plan until **shared understanding**: every major decision branch is either resolved or explicitly deferred with an owner and revisit trigger.

## How to grill

1. **Build a decision tree** — List the open forks (scope, architecture, data model, rollout, risks). Order by **dependency** (blocking decisions first).

2. **One branch at a time** — For each fork: state the tradeoff, ask the user to pick or refine, then **lock** the answer in one sentence before moving on.

3. **Your recommendation** — For each question, give a **concrete recommended answer** and why; still force the user to confirm or override.

4. **Evidence over opinion** — If the answer lives in the repo (patterns, prior art), explore the codebase and cite what you find instead of debating in the abstract.

5. **Stopping conditions** — Stop the grilling pass when:
   - All listed branches are resolved or deferred with written rationale, **or**
   - The user says **stop** / **good enough**, **or**
   - You hit **~8–12** substantive decisions in one session (summarize and offer to continue).

## Output shape

End each round with:

- **Resolved** — bullet list of decisions + one-line rationale each
- **Open / deferred** — what remains, who decides, by when (if stated)
- **Risks** — top 2–3 ways the plan could still fail

Interview the user relentlessly until those stopping conditions are met: one branch at a time, recommendations included, codebase used when it can answer.
