---
name: to-issues
description: Break a plan, spec, or PRD into independently-grabbable issues using tracer-bullet vertical slices. Use when user wants to convert a plan into issues, create implementation tickets, or break down work into issues.
---

# To Issues

Break a plan into independently-grabbable issues using vertical slices (tracer bullets).

## Process

### 1. Gather context

Ask the user for the PRD (or spec) file path (e.g. `./docs/prd/feature-name.md`).
If the PRD is not already in your context window, read it from the file.

**Convention:** Prefer PRDs under `./docs/prd/` so issue files can link back predictably.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories this addresses (if the source material has them)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Create the issue files

For each approved slice, create a numbered markdown file in `./docs/issues/{prd-name}/`, using kebab-case filenames.
Create files in dependency order (blockers first) so you can reference real filenames in the "Blocked by" field.

**Parent link:** Under `## Parent`, use a **relative path from the issue file to the PRD** (or spec), not a hardcoded guess. After you know both paths, compute it (e.g. from `./docs/issues/my-feature/01-auth.md` to `./docs/prd/my-feature.md` → `../../prd/my-feature.md`). If the PRD lives outside `./docs/prd/`, still compute the correct relative path from the issue file.

<issue-template>
## Parent

[Short label for the feature or spec](../path/from/this/issue/to/source.md)

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- Blocked by [slice-name](./01-slice-name.md) (if any)

Or "None - can start immediately" if no blockers.

</issue-template>

Do NOT modify the parent PRD file.
