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

Invoke Crucible right after you finish vibe coding a feature or refactor—before opening a pull request.

### Sample Prompts

Burn down your current working diff:

```sh
/crucible clean up my new user auth feature
```

Or simply:

```sh
/crucible
```

You can also narrow scope to a specific path or module. By default, Crucible targets active `git diff` changes. Passing an explicit file or path in your prompt allows you to run Crucible's burn-down passes on existing, unmodified files in your repository.

```sh
/crucible @/src/services/billing.tsx
```

You can also stop early:

```sh
stop crucible
```

### What Happens Next

Crucible targets your git diff by default (or an explicit path you name), strips away dead code and redundant wrappers, and verifies every change against your test suite until the scoped set is lean and review-ready.

```ts
// Before Crucible: Dead functions and unused helpers left over from prototyping
function formatId(id: string) { return id.trim() }
export function load(id: string) { return db.get(id) }

// After Crucible: Unused surface burned away
export function load(id: string) { return db.get(id) }
```

## Skill Synergy

If [ponytail](https://github.com/DietrichGebert/ponytail) is installed, crucible
leans on its YAGNI ladder for take-vs-skip. This is optional.

## License

[MIT](LICENSE).
