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

When a finding is borderline, compare it with the matching pair under Examples below.

### 5. Report

Keep it under 400 words. Group by the three headings above, and skip any heading with no findings. For each finding, give:

- `file:line`
- the check name, e.g. "possible Single-use abstraction"
- a short quote of the hunk
- the fix in one line, with the shorter code when it helps

End with one line: the finding count per heading, and the single change that would simplify the PR most. If the diff is already minimal, say so in one line and stop.

## Examples

One flagged/simpler pair per check, in TypeScript on Node. Each ends with when the pattern is fine, which settles most borderline calls.

### Simplicity first

#### Speculative feature

Asked: "Archive a workspace's records when it is deleted."

```ts
// Flagged: restore and a dry-run flag nobody asked for
export async function archiveRecords(workspaceId: string, opts: { dryRun?: boolean } = {}) { /* … */ }
export async function restoreRecords(workspaceId: string) { /* … */ }

// Simpler
export async function archiveRecords(workspaceId: string) { /* … */ }
```

Fine when the request or ticket names the extra behaviour.

#### Single-use abstraction

```ts
// Flagged: an interface, a class, and a factory around one RPC call with one caller
interface InvoiceFetcher { fetch(id: string): Promise<Invoice | undefined> }
class BillingInvoiceFetcher implements InvoiceFetcher {
  fetch(id: string) { return sendRpc(USER_ID, 'billing', 'getInvoice', { id }); }
}
export const createInvoiceFetcher = (): InvoiceFetcher => new BillingInvoiceFetcher();

// Simpler
export const fetchInvoice = (id: string) => sendRpc(USER_ID, 'billing', 'getInvoice', { id });
```

Fine when a second adapter exists today, such as a test fake swapped in at that seam.

#### Unasked flexibility

```ts
// Flagged: every caller passes 100 and 'desc'
function listInvoices(customerId: string, pageSize = 100, order: 'asc' | 'desc' = 'desc', retries = 3) { /* … */ }

// Simpler
function listInvoices(customerId: string) { /* 100 per page, newest first */ }
```

Fine when at least two callers already pass different values.

#### Impossible-case handling

```ts
// Flagged: the type says workspaceId is a string, and the caller has just read it from the record
function workspaceKey(workspaceId: string) {
  if (!workspaceId) return undefined;
  try {
    return `invoice:${workspaceId}`;
  } catch {
    return undefined;
  }
}

// Simpler
const workspaceKey = (workspaceId: string) => `invoice:${workspaceId}`;
```

Fine at a real edge: RPC payloads, Mongo documents, env vars, and user input, where the type is a promise rather than a fact.

#### Bloat

```ts
// Flagged
const paid: Invoice[] = [];
for (const invoice of invoices) {
  if (invoice.status === 'paid') {
    if (invoice.amountUsd > 0) paid.push(invoice);
  }
}
let total = 0;
for (const invoice of paid) total += invoice.amountUsd;

// Simpler
const total = invoices
  .filter(i => i.status === 'paid' && i.amountUsd > 0)
  .reduce((sum, i) => sum + i.amountUsd, 0);
```

Fine when the longer form matches the file's own style, or a step needs its own log line.

### Surgical changes

#### Drive-by change

Asked: "Log the workspace id when the invoice fetch fails."

```diff
-export async function fetchInvoice(id, workspaceId) {
-    try { return await billingRpc('getInvoice', { id, workspaceId }); }
-    catch (e) { log.error('fetch failed', e); }
+export const fetchInvoice = async (stripeInvoiceId: string, workspaceId: string) => {
+  try {
+    return await billingRpc('getInvoice', { stripeInvoiceId, workspaceId });
+  } catch (error) {
+    log.error(`fetchInvoice failed for workspace ${workspaceId}:`, error);
+  }
```

Flagged: the arrow function, renames, types, and indentation are outside the ask. Simpler: change only the log line.

Fine when the ticket asks for the cleanup, or the touched line must change anyway.

#### Style drift

```ts
// Surrounding file: named exports, async/await, errors logged and swallowed
export async function syncContact(account: Account) { /* … */ }

// Flagged: a default-exported class with promise chains and thrown errors
export default class DealSyncer {
  run(invoice: Invoice) {
    return findDeal(invoice).then(deal => { if (!deal) throw new Error('no deal'); /* … */ });
  }
}

// Simpler: match the file
export async function syncDeal(invoice: Invoice) {
  const deal = await findDeal(invoice);
  if (!deal) return log.warn(`no Deal for ${invoice.id}`);
  /* … */
}
```

Fine when the repo's documented standard differs from the file. The standard wins.

#### Own leftovers

```diff
-import { fetchLatestInvoice, fetchPreviousInvoice } from '../billing/client.js';
+import { fetchInvoices, fetchLatestInvoice, fetchPreviousInvoice } from '../billing/client.js';
-  const previous = await fetchPreviousInvoice(invoice.id, sub, workspaceId);
+  const all = await fetchInvoices(workspaceId);
```

Flagged: this diff removed the last call to fetchPreviousInvoice but kept its import. Simpler: drop it from the import.

Fine when other code still calls it. Check before removing.

#### Unasked cleanup

```diff
 export function renewalStatus(row: Row) { /* the change */ }
-
-// Old chain helper, unused since 1.0.40
-function legacyWalkBack(row: Row) { /* … */ }
```

Flagged: the dead helper predates this diff and nobody asked to remove it. Simpler: restore it, and mention it in the PR as a follow-up.

Fine when the ticket is the cleanup, or this diff is what made it dead.

### Goal-driven execution

#### Unproven fix

Bug: "A Renewal row gets both prorations merged twice."

```diff
-  await mergeProration(row, first); await mergeProration(row, second);
+  await mergeProrations(row, [first, second]);
```

Flagged: no test reproduces the double merge. Simpler: add the test first, see it fail, then fix.

```ts
it('merges two prorations into one row once', async () => {
  await syncInvoices([period, proration1, proration2]);
  expect(amountOf(rowFor(period))).toBe(period.amountUsd + proration1.amountUsd + proration2.amountUsd);
});
```

Fine when the code has no tests and the fix is a config value or a one-line typo. Say so in the PR.

#### Unguarded refactor

```diff
-export async function syncInvoice(invoice: Invoice) { /* 120 lines */ }
+export async function syncInvoice(invoice: Invoice) {
+  const period = await findPeriod(invoice);
+  return period ? updatePeriod(period, invoice) : createPeriod(invoice);
+}
```

Flagged: behaviour should be unchanged, but no test covers syncInvoice. Simpler: point to the tests that cover it, or add one per branch before splitting.

Fine when existing tests already exercise every branch. Name them in the review.
