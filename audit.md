# mem20 Audit

Shared audit log for the **mem20** project. This repository is the group's shared code storage for the audit; all assets come solely from the existing [`mem20`](https://github.com/bythewayz66-glitch/mem20) repo and whatever is created here in `mem20-audit`.

## How to contribute

**Each contributor appends their own section to this same file.** Do not create a separate file and do not rewrite or reorder anyone else's section — add yours at the bottom.

1. Append a new section at the end of this file, using the template below.
2. Use a level-2 heading with your name/handle, e.g. `## <your-name>`.
3. Keep your findings, evidence and open questions inside your own section.
4. Commit to a branch and open a pull request against `main`.

### Section template

```markdown
## <your-name>

- **Scope:** what part of mem20 you audited
- **Findings:** what you found (with file paths / links as evidence)
- **Severity:** low / medium / high
- **Open questions:** anything unresolved
```

---

## Sections

<!-- Append your own section below this line. One section per contributor. -->

## Git Workflow Master

- **Scope:** Git workflow, branch/merge discipline and merge-gate enforcement for `mem20` (read-only source of truth) — branch inventory, PR/merge state, `main` protection, the `pr-gate.yml` two-tier gate, and the `mem20-audit` contribution flow itself. All numbers below are live reads taken 2026-11-06, not from the brief.
- **Findings:**

  1. **`main` is unprotected — Rule 0 is policy-only, unenforced.** `docs/git/README.md` states "Rule 0 — nobody commits to `main`", but the branch has no protection and no ruleset: `GET /repos/bythewayz66-glitch/mem20/branches/main/protection` → **404**, `GET .../rulesets` → `[]`, `GET .../rules/main` → `[]`. Nothing stops a direct push, a force-push, or a merge with the gate red.

  2. **The gate is well-designed but is not a required check.** `.github/workflows/pr-gate.yml` defines `hygiene` (diff-scoped, "required from day one") and `suite` (advisory until repo variable `MEM20_REQUIRE_GREEN_SUITE=1`). Because `main` has no protection, **neither job is a required status check** — a PR can be merged while `hygiene` is failing. The gate only bites if it is wired into branch protection.

  3. **Branch sprawl: 98 branches, only `main` exists as a protected ref.** 40 branches carry a **merged** PR (safe to delete after owner approval); **48 branches have no PR at all** (drafts/orphans), e.g. `tmp/cpp-expert-commit-probe`, `tmp/cpp-expert-push-probe`, `token-directive-confirm-1793572016`, `feat/4.0-server-builder-mem20-mcp`, `fix/post20-defects`. Prefix distribution: `feat` 33, `docs` 25, `fix` 15, `dependabot` 6, `chore` 4, `tmp` 2 — plus off-taxonomy prefixes (`deck/`, `gtm/`, `launch/`, `pricing/`, `sdk/`, `opencode/`, `ops/`) that are not in the documented prefix table.

  4. **Merge method is `--no-ff`, not the documented rebase default.** The last 100 commits contain **32 merge commits** (e.g. `b9c8f7ea Merge fix/retire-kill-switch`, `4e5903cc Merge fix/toolchest-inventory: close #228`, `1980d2b9 Merge fix/ops-dreamz: close #234 and #238`). Not wrong, but the policy's "default rebase" is also unenforced and the first-parent line is noisy.

  5. **Open PRs: 9** — 6 dependabot (`#261`–`#266`), `#260 fix/open-issues-5`, `#258 opencode/absolute-truth-113`, `#241 docs/agentz-evidence`. **33 PRs were closed without merging.** The 6 dependabot PRs carry no workstream label and sit open with no merge/close decision.

  6. **Issue-label taxonomy is too thin to route work.** 193 non-PR issues (105 open / 88 closed) but only **8 distinct labels** in use: `bug` 42, `severity:high` 6, `severity:medium` 5, `documentation` 5, `severity:low` 2, `ci` 1, `test-failure` 1, `docs` 3. There is **no workstream label set and no `unowned`/`needs-triage`**, so the policy's "one workstream label per issue" rule has nothing to route on. The 105 open issues are dominated by the auto-filed `conformance: <subsystem> fails criterion 5,7` series (`#292`–`#391`) with **no labels at all**.

  7. **`mem20-audit` itself is unprotected and the flow is unexercised.** `main` has no protection (protection endpoint 401/absent), 0 PRs, 0 issues, single linear history (5 commits: `1441fe70` → `a3f4b920` → `ea14e75d` → `c834d66e` → `b0ce433c`). The documented flow (branch `audit/<name>` → PR → one reviewer merges) is correct but has never run — this is the first PR. `delete_branch_on_merge` is not set, so merged audit branches will accumulate.

  8. **Recovery rules are stated but unenforced.** The policy correctly bans force-push to `main` and prefers `--force-with-lease`; with no protection, nothing enforces it.

- **Recommendations (fix/complete, never remove):**
  - Enable branch protection / a ruleset on `mem20` `main`: require a PR, require the `hygiene` check, require 1 approval, dismiss stale approvals, block force-push and deletion. **Owner action** — needs repo admin; the available token cannot set it (protection GET returns 404/401).
  - Arm `suite` (`MEM20_REQUIRE_GREEN_SUITE=1`) **only after** WS-2 item 1 lands, per the policy — do not arm early.
  - Set `delete_branch_on_merge=true` on both repos.
  - Triage the 48 no-PR branches and 40 merged-PR branches into keep/split/delete; **deletion needs the owner's written approval**.
  - Create the workstream label set plus `unowned`/`needs-triage`, and label the 105 open issues (at minimum the conformance series).
  - Decide the 6 open dependabot PRs (merge or close).

- **Severity:** high — an unprotected default branch plus an unenforced gate on a repo with 98 branches and 105 open issues.
- **Open questions:**
  - Who owns the branch-protection change (requires repo admin)?
  - Is `--no-ff` the intended merge default, or should the policy's "default rebase" be enforced?
  - Are any of the 48 orphan branches still live work, or can they be deleted?
