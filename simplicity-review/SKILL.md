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

Asked: "Send a welcome email when a user signs up."

```ts
// Flagged: templates, scheduling, and a dry-run flag nobody asked for
export async function sendWelcome(user: User, opts: { template?: string; sendAt?: Date; dryRun?: boolean } = {}) { /* … */ }

// Simpler
export async function sendWelcome(user: User) { /* … */ }
```

Fine when the request or ticket names the extra behaviour.

#### Single-use abstraction

```ts
// Flagged: an interface, a class, and a factory around one HTTP call with one caller
interface WeatherSource { today(city: string): Promise<Forecast> }
class HttpWeatherSource implements WeatherSource {
  async today(city: string) { return (await fetch(`${API}/forecast?city=${city}`)).json(); }
}
export const createWeatherSource = (): WeatherSource => new HttpWeatherSource();

// Simpler
export const fetchForecast = async (city: string): Promise<Forecast> =>
  (await fetch(`${API}/forecast?city=${city}`)).json();
```

Fine when a second adapter exists today, such as a test fake swapped in at that seam.

#### Unasked flexibility

```ts
// Flagged: every caller passes 50 and 'desc'
function listOrders(userId: string, pageSize = 50, order: 'asc' | 'desc' = 'desc', retries = 3) { /* … */ }

// Simpler
function listOrders(userId: string) { /* 50 per page, newest first */ }
```

Fine when at least two callers already pass different values.

#### Impossible-case handling

```ts
// Flagged: the type says id is a string, and the caller has just read it from a typed object
function cacheKey(id: string) {
  if (!id) return undefined;
  try {
    return `order:${id}`;
  } catch {
    return undefined;
  }
}

// Simpler
const cacheKey = (id: string) => `order:${id}`;
```

Fine at a real edge: HTTP payloads, database documents, env vars, and user input, where the type is a promise rather than a fact.

#### Bloat

```ts
// Flagged
const shipped: Order[] = [];
for (const order of orders) {
  if (order.status === 'shipped') {
    if (order.total > 0) shipped.push(order);
  }
}
let revenue = 0;
for (const order of shipped) revenue += order.total;

// Simpler
const revenue = orders
  .filter(o => o.status === 'shipped' && o.total > 0)
  .reduce((sum, o) => sum + o.total, 0);
```

Fine when the longer form matches the file's own style, or a step needs its own log line.

### Surgical changes

#### Drive-by change

Asked: "Log the user id when loading the profile fails."

```diff
-export async function loadProfile(id) {
-    try { return await api.get('/profile/' + id); }
-    catch (e) { log.error('load failed', e); }
+export const loadProfile = async (userId: string) => {
+  try {
+    return await api.get(`/profile/${userId}`);
+  } catch (error) {
+    log.error(`loadProfile failed for user ${userId}:`, error);
+  }
```

Flagged: the arrow function, renames, types, and indentation are outside the ask. Simpler: change only the log line.

Fine when the ticket asks for the cleanup, or the touched line must change anyway.

#### Style drift

```ts
// Surrounding file: named exports, async/await, errors logged and swallowed
export async function syncUser(user: User) { /* … */ }

// Flagged: a default-exported class with promise chains and thrown errors
export default class CartSyncer {
  run(cart: Cart) {
    return findOwner(cart).then(owner => { if (!owner) throw new Error('no owner'); /* … */ });
  }
}

// Simpler: match the file
export async function syncCart(cart: Cart) {
  const owner = await findOwner(cart);
  if (!owner) return log.warn(`no owner for cart ${cart.id}`);
  /* … */
}
```

Fine when the repo's documented standard differs from the file. The standard wins.

#### Own leftovers

```diff
-import { getOrder, getOrderItems } from './orders.js';
+import { getOrder, getOrderItems, getOrderWithItems } from './orders.js';
-  const items = await getOrderItems(order.id);
+  const { items } = await getOrderWithItems(order.id);
```

Flagged: this diff removed the last call to getOrderItems but kept its import. Simpler: drop it from the import.

Fine when other code still calls it. Check before removing.

#### Unasked cleanup

```diff
 export function orderLabel(order: Order) { /* the change */ }
-
-// Old formatter, unused since v2
-function legacyLabel(order: Order) { /* … */ }
```

Flagged: the dead helper predates this diff and nobody asked to remove it. Simpler: restore it, and mention it in the PR as a follow-up.

Fine when the ticket is the cleanup, or this diff is what made it dead.

### Goal-driven execution

#### Unproven fix

Bug: "A discount gets applied twice when two coupons arrive together."

```diff
-  await applyCoupon(cart, first); await applyCoupon(cart, second);
+  await applyCoupons(cart, [first, second]);
```

Flagged: no test reproduces the double discount. Simpler: add the test first, see it fail, then fix.

```ts
it('applies two coupons once each', async () => {
  const cart = await checkout({ items: [book], coupons: [tenOff, fiveOff] });
  expect(cart.total).toBe(book.price - 10 - 5);
});
```

Fine when the code has no tests and the fix is a config value or a one-line typo. Say so in the PR.

#### Unguarded refactor

```diff
-export async function placeOrder(cart: Cart) { /* 120 lines */ }
+export async function placeOrder(cart: Cart) {
+  const stock = await reserveStock(cart);
+  return stock.ok ? charge(cart) : rejectOrder(cart, stock.missing);
+}
```

Flagged: behaviour should be unchanged, but no test covers placeOrder. Simpler: point to the tests that cover it, or add one per branch before splitting.

Fine when existing tests already exercise every branch. Name them in the review.
