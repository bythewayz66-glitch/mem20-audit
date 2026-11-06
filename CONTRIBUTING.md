# Contributing to the mem20 Audit

This repository is the shared code storage for the group audit of the **mem20** project.

## Two hard rules

### 1. `audit.md` is the ONLY shared file

[`audit.md`](audit.md) is the single shared document. Every contributor appends **their own
section** to the bottom of it.

- ✅ Append a new `## <your-name>` section at the **end** of `audit.md`.
- ❌ Do **not** create a second shared file, and do **not** create a separate file for your findings.
- ❌ Do **not** rewrite, reorder, reformat or delete anyone else's section — not even to fix a typo.
- ❌ Do **not** insert your section in the middle. The file is append-only so the audit reads as one thread.

If you need to correct your own earlier section, append a new dated note under your own
heading instead of editing history in place.

### 2. `mem20` is read-only

The [`mem20`](https://github.com/bythewayz66-glitch/mem20) repo is the **read-only source of
truth**. It is the input to this audit, never an output.

- ❌ Never push, commit, open a PR against, or otherwise modify `mem20`.
- ❌ Never open issues or comments on `mem20` as part of this audit.
- ✅ Read it, clone it, cite it — then write your findings here.

### Everything else is yours

Apart from `audit.md` (shared, append-only) and `mem20` (read-only), you may create whatever
you want inside `mem20-audit` — scripts, notes, repro cases, tooling, scratch directories.
Use your own directory or a clearly named file so you don't collide with anyone else.

## Section format

Use a level-2 heading with your name or handle, then this template:

```markdown
## <your-name>

- **Scope:** what part of mem20 you audited
- **Findings:** what you found (with file paths / links as evidence)
- **Severity:** low / medium / high
- **Open questions:** anything unresolved
```

## PR steps

1. **Fork or branch.** Either fork this repo, or — if you already have write access —
   create a branch: `git checkout -b audit/<your-name>`.
2. **Append your section** to the bottom of `audit.md`. Keep the diff to your own added lines.
3. **Open a pull request against `main`.** Title it `audit: <your-name> section`. In the
   body, state your scope and severity in one or two lines.
4. **One reviewer merges.** A single reviewer approves and merges the PR. Do not merge your
   own PR, and do not push directly to `main`.

### Keeping your branch current

If `main` has moved on since you branched, rebase onto it before opening the PR so your
append lands cleanly at the bottom:

```bash
git fetch origin
git rebase origin/main
```

If a conflict appears, keep **both** sections — yours goes last.

## Review checklist (for the reviewer)

- [ ] The PR only appends to `audit.md` — no edits to existing sections.
- [ ] The new section sits at the bottom and uses the `## <name>` heading.
- [ ] Findings cite evidence (file paths or links into `mem20`).
- [ ] Severity is stated.
- [ ] Nothing in the PR touches the `mem20` repo.

## Access

Write access is granted by adding a GitHub handle as a collaborator on this repo. If you do
not have write access, fork and open a PR — that works with no collaborator invite at all.
