---
name: to-prd
description: Turn the current conversation context into a PRD and save it under ./docs/prd. Use when user wants to create a PRD from the current context.
---

This skill takes the current conversation context and codebase understanding and produces a PRD.

**No broad interview:** do not run a discovery Q&A from scratch or repeat questions already answered in the thread. Synthesize what is already in context.

**Narrow confirmations allowed:** you may ask at most **one short checkpoint** before writing—e.g. “Do these module boundaries look right?” and “Which modules should get tests?”—only if the thread leaves that ambiguous. If the user already stated modules or testing intent, skip the checkpoint.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already.

2. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

Apply the **narrow confirmations** rule above for module list and test scope.

3. Write the PRD using the template below and save it as a markdown file under `./docs/prd/` (create the directory if it doesn't exist). Use a descriptive kebab-case filename (e.g. `./docs/prd/yacht-search-filters.md`).

Section headings **Implementation Decisions**, **Testing Decisions**, **Out of Scope**, and **Further Notes** intentionally mirror [request-refactor-plan/SKILL.md](../request-refactor-plan/SKILL.md) so plans and PRDs stay consistent; keep the same semantics when filling them.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
