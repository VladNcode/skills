---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change; stable tests describe what callers and users can rely on.

Prefer integration-style tests that exercise real paths through public APIs. Avoid coupling to internals (private methods, internal collaborators, or verifying via storage/query shortcuts). When a refactor changes structure but not behavior, tests should still pass—if they fail, they were likely specifying implementation, not behavior.

Details and examples: [tests.md](tests.md). Where to mock: [mocking.md](mocking.md).

## Anti-Pattern: Horizontal Slices

**Do not** write the full test suite, then the full implementation (“horizontal” RED/GREEN). That yields tests for imagined behavior and brittle shape-checking instead of observed behavior.

**Prefer** vertical slices (tracer bullets): one failing test, minimal code to pass, repeat. Each cycle incorporates what you learned from the last.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Workflow

### 1. Planning

Before writing any code:

- [ ] Agree on public interface changes and which behaviors to test (prioritized—you cannot cover everything)
- [ ] Note [deep-module](deep-modules.md) opportunities and [testable interface](interface-design.md) shapes
- [ ] Get approval on the plan

Ask: “What should the public interface look like? Which behaviors matter most?”

### 2. RED / GREEN loop

For **each** behavior (the first one is your tracer bullet; the rest are the same pattern):

```
RED:   one new failing test for one behavior
GREEN: minimal code until that test passes
```

Rules: one test at a time, no speculative features, assert observable outcomes through the public surface.

### 3. Refactor

With the suite green, use [refactoring.md](refactoring.md) as the scan list. **Never refactor while RED.**

## Checklist Per Cycle

```
[ ] Test names and assertions describe behavior, not internals
[ ] Exercise the public interface; no “how” coupling
[ ] Code is minimal for the current test only
```
