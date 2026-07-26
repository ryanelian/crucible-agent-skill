# Crucible

[![skills.sh](https://skills.sh/b/ryanelian/crucible-agent-skill)](https://skills.sh/ryanelian/crucible-agent-skill)

**Less code is more. Code that does not exist cannot slow down your app. And it cannot slow down your team.**

You spend twenty minutes vibe coding a new feature with an AI assistant, and it works flawlessly on the first try. But when you finally run git diff, you are greeted by a 2,000-line monster diff. Getting the feature to work was lightning fast, but making it maintainable is a whole different challenge.

Crucible is designed to solve that exact pain point by acting as an automated burn-down pass for your diff. Right after you finish vibe coding, Crucible takes over and runs continuous, guarded loops over your modified files to prune away the noise. It systematically deletes dead exports, flattens pass-through functions, and cleans up unneeded bloat.

To ensure your code never breaks, Crucible applies strict guardrails to verify tests or compile-time checks after every tweak.

This guarantees your pull request is actually maintainable for your team. By the time Crucible completes its run, your diff is lean, readable, and ready for code review.

Burn the diff down. Optimize, simplify, and delete dead code, pass after pass, until a clean pass finds nothing worth taking. Runs in batches of 10 passes.

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
