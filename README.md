# Crucible

[![skills.sh](https://skills.sh/b/ryanelian/crucible-agent-skill)](https://skills.sh/ryanelian/crucible-agent-skill)

**Less code is more. Code that doesn't exist can't slow down your app — or your engineering team.**

AI-assisted coding has unlocked unprecedented developer velocity. Prototyping a feature now takes minutes instead of days. But that speed creates a new bottleneck: **massive pull requests.**

A 2,000-line diff might work on your machine, but it exhausts reviewer bandwidth, blurs architectural boundaries, and quietly compounds technical debt.

**Just as a real-life crucible liquefies crude ore to isolate pure metal from molten waste, Crucible burns away unrefined vibe code to leave only lean, production-grade logic.** 

* **Autonomous Burn-Down:** Runs continuous, guarded loops over your modified files right after you finish building to strip away noise.
* **Systematic Refinement:** Eliminates dead code, flattens pass-through abstractions, enforces strict layer boundaries (**Sink & Lift**), and trims architectural bloat.
* **Test-Guarded Safety:** Verifies every change against your test suite and compiler checks before accepting it to guarantee zero regressions.

> **Proven Production Impact:** Tested in real-world production environments, Crucible slashed a **2,000-line diff down to 1,000 lines (a 50% reduction)** while significantly sharpening Single Responsibility Principle (SRP) boundaries. *(Results vary by codebase, but diff bloat elimination is guaranteed!)*

## Install

```bash
npx skills add ryanelian/crucible-agent-skill
```

Or install only this skill / globally / list:

```bash
npx skills add ryanelian/crucible-agent-skill --skill crucible
npx skills add ryanelian/crucible-agent-skill -g
npx skills add ryanelian/crucible-agent-skill --list
```

Local checkout:

```bash
npx skills add ./crucible-agent-skill --skill crucible
```

Package page: [skills.sh/ryanelian/crucible-agent-skill](https://skills.sh/ryanelian/crucible-agent-skill)  
Source: [github.com/ryanelian/crucible-agent-skill](https://github.com/ryanelian/crucible-agent-skill)

## Usage

After a feature or messy refactor:

- `crucible` / `/crucible` — loop until empty pass; ask after each 10-pass batch
- `refine until clean` / `burn down diff` / `simplify until done`
- `crucible on path/or/area` — narrow scope

Scope is that feature’s diff: **staged + unstaged + relevant untracked**. If the
working tree is clean: `git diff <default-branch>...HEAD` (`main` / `master` /
`origin/HEAD` / etc.). Edits stay in that set; export and hop-deletion caller
checks are **repo-wide**.

Off: `stop crucible` / `normal mode`.

Every pass must print its number. Cap is 10 per batch:

```
Pass 1: took [dead formatId]. skipped [rename churn].
Pass 2: took [inline fetchUser]. skipped [].
Pass 3: nothing worth taking. Crucible done.
```

At a non-empty ceiling:

```
Pass 10: took [dead branch]. skipped []. Cap reached. Continue for another 10 passes?
```

Yes → another `Pass 1`…`Pass 10` batch (ask again). No → done. Empty pass → done, no ask.

Ambiguous skips are tagged and rolled up when the run stops or hits a cap:

```
Pass 1: took []. skipped [ambiguous: ErrorCode map — might be public API].
…
Pass 3: nothing worth taking. Crucible done.
Leftover ambiguous: [ErrorCode map — might be public API].
```

### YAGNI 

For example, Crucible will delete a dead helper function with zero callers:

```ts
// before
function formatId(id: string) { return id.trim() }
export function load(id: string) { return db.get(id) }

// after
export function load(id: string) { return db.get(id) }
```

## Pairing

If [ponytail](https://github.com/DietrichGebert/ponytail) is installed, crucible
leans on its YAGNI ladder for take-vs-skip. This is optional.

## Layout

```
skills/
└── crucible/
    └── SKILL.md
```

## License

[MIT](LICENSE).
