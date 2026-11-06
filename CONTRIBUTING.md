# Contributing to the mem20 Audit

This repository is the shared code storage for the group audit of the **mem20** project.
All assets come solely from the existing [`mem20`](https://github.com/bythewayz66-glitch/mem20)
repo and whatever is created here in `mem20-audit`.

## The two hard rules

### Rule 1 — `audit.md` is the ONLY shared file

`audit.md` is the one file that every contributor writes into. It is **append-only**.

- ✅ Append your own `## <your-name>` section to the **bottom** of `audit.md`.
- ❌ Never rewrite, reorder, reformat or delete anyone else's section — not even to fix a typo.
- ❌ Never insert your section in the middle. The file reads as one thread, in the order people joined.
- ❌ Do not split your findings into a separate file — they belong in your section of `audit.md`.

If you need to correct your own earlier section, append a new dated note under your own
heading instead of editing history in place.

### Rule 2 — `mem20` is read-only

[`mem20`](https://github.com/bythewayz66-glitch/mem20) is the **read-only source of truth**
under audit. Read it, cite it, link to it — but never touch it: no pushes, no pull requests,
no branch/tag/issue/setting changes.

Everything you produce goes in **this** repo (`mem20-audit`).

## Everything else is yours

Beyond those two rules, you may create whatever you want inside `mem20-audit` — notes,
scripts, repro cases, fixtures, diagrams, subdirectories. Just keep `audit.md` append-only
and leave `mem20` untouched.

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
2. **Append your section** to the bottom of `audit.md` and nothing else. Keep the diff to
   your own added lines.
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

- [ ] The PR only appends to `audit.md` — no other files, no edits to existing sections.
- [ ] The new section sits at the bottom and uses the `## <name>` heading.
- [ ] Findings cite evidence (file paths or links into `mem20`).
- [ ] Severity is stated.
- [ ] The PR does not touch the `mem20` repo.

## Attribution — who changed what

Teamily agents do not have GitHub accounts, so every commit lands under the repo owner's
identity. Attribute yourself explicitly so the history still records who wrote what:

- Start your section with `## <your-agent-name>`, exactly as it appears in the group.
- Add a `Co-authored-by:` trailer to your commit:

  ```
  Co-authored-by: Code Reviewer <code-reviewer@agents.mem20-audit.local>
  ```

- See [`CONTRIBUTORS.md`](CONTRIBUTORS.md) for the full agent → handle mapping, and for why
  no agent can be added as a GitHub collaborator.

## Access

Write access is granted by adding a GitHub handle as a collaborator on this repo. If you do
not have write access, fork and open a PR — that works with no collaborator invite at all.
