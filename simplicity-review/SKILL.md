---
name: simplicity-review
description: Review a PR or branch diff for over-engineering and noisy changes, using the Karpathy-inspired guidelines (simplicity first, surgical changes, goal-driven execution). Flags speculative code, drive-by edits, style drift, leftovers, impossible-case handling, bloat, and unproven fixes. Use when the user wants a simplicity / over-engineering / "is this too much" review of a PR, branch, or work-in-progress changes.
---

Review the diff between `HEAD` and a fixed point for one question: **is this the smallest clean change that does the job?**

It does not hunt for bugs or check the spec in depth. Use `/code-review` for that.

## Process

### 1. Pin the diff

- PR number or URL → `gh pr view <pr> --json title,body,baseRefName,headRefName` and `gh pr diff <pr>`.
- Branch, commit, or tag → `git diff <fixed-point>...HEAD` and `git log <fixed-point>..HEAD --oneline`.
- Nothing given → ask for the fixed point.

Fail early on a bad ref or an empty diff.

### 2. Pin the request

Every changed line must trace back to what was asked. Find the ask, in this order:

1. The PR title and body.
2. Issue references in commits or the PR body, fetched via `docs/agents/issue-tracker.md` if present.
3. A spec under `docs/`, `specs/`, or `.scratch/` that matches the branch name.
4. Commit messages, as a last resort. Say so in the report.

### 3. Read the surroundings

For each changed file, read enough of the unchanged code to know its local style and patterns. Style drift and bloat are judged against the file itself, not general taste.

The repo's documented standards (`CLAUDE.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`) override every check below. Skip anything a linter or formatter already enforces.

### 4. Run the checks

Each check is a judgement call. Label it as "possible …", never as a hard violation.

**Simplicity first:** the minimum code that solves the problem, with nothing speculative.

- **Speculative feature**: behaviour the request didn't ask for. → cut it, or split it into its own PR.
- **Single-use abstraction**: a helper, class, interface, or factory with one caller and no real need. → inline it.
- **Unasked flexibility**: options, params, config, or hooks nobody uses yet. → hard-code the one case.
- **Impossible-case handling**: try/catch, null checks, or fallbacks for states the types or callers rule out. → delete it, or say why the state is reachable.
- **Bloat**: the same behaviour fits in far fewer lines. → show the shorter version.

**Surgical changes:** touch only what you must, and clean up only your own mess.

- **Drive-by change**: reformatting, renames, comment edits, or refactors outside the task. → revert them, or move them to a separate PR.
- **Style drift**: new code ignores the patterns around it (naming, error handling, comment density, idioms). → match the file.
- **Own leftovers**: imports, variables, or functions this diff made unused but kept. → remove them.
- **Unasked cleanup**: pre-existing dead code deleted without being asked. → restore it and mention it instead.

**Goal-driven execution:** define success, then prove it.

- **Unproven fix**: a bug fix with no test that reproduces the bug. → add a failing-then-passing test.
- **Unguarded refactor**: behaviour-preserving changes with no tests covering the touched code. → add or point to tests.

### 5. Report

Keep it under 400 words. Group by the three headings above, and skip any heading with no findings. For each finding, give:

- `file:line`
- the check name, e.g. "possible Single-use abstraction"
- a short quote of the hunk
- the fix in one line, with the shorter code when it helps

End with one line: the finding count per heading, and the single change that would simplify the PR most. If the diff is already minimal, say so in one line and stop.
