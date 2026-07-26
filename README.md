# Crucible

[![skills.sh](https://skills.sh/b/ryanelian/crucible-agent-skill)](https://skills.sh/ryanelian/crucible-agent-skill)

**Mantra:** Less code is more. Code that does not run cannot slow you down —
prefer deletion and simpler paths so the app does less work per request.

Burn a diff down. Optimize, simplify, delete dead code — pass after pass — until
a clean pass finds nothing worth taking. Batches of **10 passes**; after a full
non-empty batch, ask whether to run another 10.

Less code only when it does not hurt performance, security, or correctness.
No new abstractions "for clarity."

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

YAGNI take — dead helper, zero callers:

```ts
// before
function formatId(id: string) { return id.trim() }
export function load(id: string) { return db.get(id) }

// after
export function load(id: string) { return db.get(id) }
```

## Pairing

If [ponytail](https://github.com/DietrichGebert/ponytail) is installed, crucible
leans on its YAGNI ladder for take-vs-skip. Optional — the gate stands alone.

## Layout

```
skills/
└── crucible/
    └── SKILL.md
```

## License

[MIT](LICENSE).
