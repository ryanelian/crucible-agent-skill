---
name: crucible
description: >
  Repeatedly refine modified code for optimization, simplification, and dead-code
  elimination until a full pass finds nothing worth taking (batches of 10 passes;
  after each full non-empty batch, ask whether to run another 10). Take a change
  only when it does not overcomplicate and does not hurt performance, security,
  or correctness. Use when the user says "crucible", "refine until clean",
  "simplify until done", "loop until no more opportunities", or invokes /crucible
  — especially after a feature/refactor when the diff should be burned down.
argument-hint: "[path|scope]"
license: MIT
---

# Crucible

**Mantra:** Less code is more. Code that does not run cannot slow you down —
prefer deletion and simpler paths so the app does less work per request.

Burn the diff down. Pass after pass. Stop on an empty pass. Batches cap at 10;
after a full non-empty batch, ask before another 10. Less code only when it does
**not** hurt performance, security, or correctness — and does not overcomplicate.

## Persistence

ACTIVE until an empty pass or the user declines another batch. Prove “done”
with an empty pass (unless the batch cap hits first).

Off: "stop crucible" / "normal mode". Narrow: `/crucible path/or/area`.

**Loop in one turn when tools allow.** Passes back-to-back until empty or cap;
no asks between passes. Yield mid-batch only if tools are blocked — then
loop/wake (`crucible`, next N + scope) or ask to continue. Never re-arm after
empty pass or declined batch.

## Pass cap

**10 passes per batch.** N resets to 1 each batch. One `Pass N:` line per pass
(see Output). Never skip a number. Never start pass 11 in a batch. Never
auto-continue past the cap.

| Stop | Next |
|------|------|
| Empty pass (any N) | Done. No ask. |
| Pass 10 non-empty | Ask for another 10. |
| User **yes** | New batch, N = 1, same scope + lists. |
| User **no** | Done. |

## Scope

Target = the feature/refactor diff — not the whole repo. User may narrow path.

Resolve (first that yields files):

1. **Dirty:** `git diff` + `git diff --staged` + untracked that belong to the
   change (skip `node_modules/`, build output, `.env`, etc.)
2. **Clean tree:** `git diff <default-branch>...HEAD` (three dots). Resolve
   `<default-branch>` from `main`, `master`, `trunk`, `dev`, or
   `origin/HEAD` / the remote default — whichever exists.
3. **Fallback:** files touched this session that belong to the change.

Lock that **edit set** for the session (plus files crucible changes inside it).
Re-read contents each pass. Do not expand edits into new modules.

**Caller checks are repo-wide.** Edits stay in the edit set; before deleting or
narrowing an export, search the whole repo for importers/callers. Zero callers
in scope is not enough.

Tests for scoped code are in scope when they block a take or assert deleted API.

## The pass

N starts at 1; increment by 1 each pass.

1. **Read** scoped files as they are now.
2. **Hunt** only for:
   - **Dead code** — unused exports/helpers/methods with zero callers
     **across the repo**
   - **Test-only production surface** — app/prod symbols whose only callers are
     tests (or test helpers). Prefer delete the prod API and fix/remove those
     tests. Skip / mark ambiguous if it looks like an intentional test seam or
     supported test utility (e.g. `testing/` export, documented testkit).
   - branches that duplicate the same return
   - single-use wrappers / temporaries that only rename
   - **Pass-through chains** — `A → B → C → D` where B/C only forward. Prefer
     `A` call the real work (`D` or inlined body). Delete empty middle hops
     **only if** they have zero other callers **repo-wide** and add no policy,
     validation, auth, retries, or observability. Skip / ambiguous if a hop
     owns real behavior or is a shared boundary.
   - equivalent shorter stdlib/API already used nearby
3. **Take** gate-passing candidates **one at a time**; verify each before the next.
4. **Verify** lightest check: scoped tests → else typecheck/lint on touched files
   → else note skipped. **On failure:** revert that candidate; do not patch it;
   record `skipped [failed verification: <short reason>]`; continue other
   candidates in this pass.
5. **Report** one line for N (see Output). Then: empty → stop; N = 10 non-empty
   → ask; N = 10 empty → stop; else → N + 1 immediately.

## Gate

**Take** = YAGNI or KISS that holds all of:

- Less code, or clearly simpler control flow
- No new layer / indirection / framework "for clarity"
- Does **not** hurt **performance** (same or better on the hot path)
- Does **not** hurt **security** (no weaker auth, validation, or trust boundaries)
- Does **not** hurt **correctness** (same behavior, including error/cancel paths)

**Skip** (do not take):

- Rename / move churn with no behavior or size win
- Layer merges/splits that need a design fight
- Clever micro-opts that add structure or a second path
- Anything that weakens security or can fail open into wrong data
- Migrations, new dependencies, broad API rewrites
- Accessibility basics or anything the user asked to keep

When unsure: **don't** — tag as ambiguous.

## Ambiguous

Unclear takes (not hard Gate skips). Session list:

- Per pass: `skipped [ambiguous: <what> — <why one clause>]`
- Dedup by subject; refresh why if it changes
- Roll up on stop / cap ask (see Output); not mandatory work

## Rules

- **YAGNI** — delete dead code / unused surface; no "might need later."
- **KISS** — fewer layers; no abstraction unless it removes more than it adds.
- **No hurt** — never trade performance, security, or correctness for line count.
- **No oscillation** — do not undo a structural change from earlier in this
  session (including prior batches).
- No new features. No drive-bys outside the edit set.
- Deletion over abstraction. Boring over clever. Gate first, aesthetic second.
- Repo-wide caller check before deleting an export; classify prod vs test for
  test-only surface.
- Failed verification → revert and skip; never fix-forward.
- Ambiguous → skip, tag, carry; never silent.
- User insists on a skipped item → take next pass, no re-arguing.

## Take vs skip

**Take (dead code)** — zero repo-wide callers:

```ts
// before
function formatId(id: string) { return id.trim() }
export function load(id: string) { return db.get(id) }

// after
export function load(id: string) { return db.get(id) }
```

`Pass 1: took [dead formatId]. skipped [].`

**Take (test-only prod surface)** — only referenced from tests:

```ts
// before: prod
export function buildFixtureUser() { return { id: "t" } }
// only importer: app.test.ts

// after: delete buildFixtureUser; drop or rewrite the test that needed it
```

`Pass 1: took [test-only buildFixtureUser + test update]. skipped [].`

**Take (KISS / flatten)** — thin wrapper or pure forwarder chain:

```ts
// before: show → fetchUser → api.get  OR  s1 → s2 → s3 → s4
// after:  show → api.get              OR  s1 → s4 (delete unused middle hops)
```

`Pass 1: took [inline fetchUser]. skipped [].`  
`Pass 1: took [flatten service1→service4; dead service2/service3]. skipped [].`

**Skip:**

```
Pass 1: took []. skipped [rename churn, extract interface, micro-opt cache].
Pass 1: took [dead formatId]. skipped [failed verification: typecheck — inline fetchUser].
Pass 1: took []. skipped [ambiguous: testing/createApp — supported testkit].
Pass 1: took []. skipped [ambiguous: service2 adds auth check — not a pure forwarder].
```

## Output

Every pass: exactly one numbered line. No essays. No silent passes.

Pattern: `Pass N: took […]. skipped […].`

Tag skips when useful: plain gate skip, `ambiguous: …`, `failed verification: …`.

Empty → done. Cap (non-empty pass 10) → ask. Decline → done. After any of those,
emit `Leftover ambiguous: […]; […].` if the list is non-empty (omit if empty).
Keep the list across yes-batches.

```
Pass 1: took [dead formatId]. skipped [rename churn, ambiguous: ErrorCode map — might be public API].
Pass 2: took [inline fetchUser]. skipped [].
Pass 3: nothing worth taking. Crucible done.
Leftover ambiguous: [ErrorCode map — might be public API].
```

Cap ask form: `Pass 10: took […]. skipped […]. Cap reached. Continue for another 10 passes?`

## Boundaries

Crucible governs deletes/simplifies in the edit set, not how you talk.
"stop crucible" / "normal mode": off. Scope ends on empty pass, declined batch,
or session end.

If **ponytail** is available, use its ladder for take vs skip. Crucible owns the
loop. No ponytail? The gate is enough.

Burn what is already in the crucible.
