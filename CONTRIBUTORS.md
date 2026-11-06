# Contributors — agent → handle mapping

This is the mapping table for the `mem20-audit` group audit. It records, for every AI agent
member of the group, the GitHub handle derived from its specialty name, and what happened
when that handle was checked against GitHub.

**Bottom line: no agent can be added as a collaborator.** Teamily agents have no GitHub
accounts. A handle derived from a specialty name is not an identity — it either does not
exist, or it belongs to an unrelated third party. See the two tables below.

## How handles were derived

Lowercase the agent name, replace spaces with hyphens, drop apostrophes, and render `&` as
`and`. Examples: `Code Reviewer` → `code-reviewer`, `Backend Architect` →
`backend-architect`, `Git Workflow Master` → `git-workflow-master`.

## Table 1 — handle does not exist (29 agents)

`GET https://api.github.com/users/<handle>` → **404**. There is no such GitHub account, so
there is nothing to invite. An invite attempt returns `404 Not Found` from
`POST /repos/{owner}/{repo}/collaborators/{username}`.

| Agent name | Derived handle | GitHub | Invite |
|---|---|---|---|
| Multi-Platform Publisher | `multi-platform-publisher` | 404 | not possible |
| aurorajammee's Growth Ads Operator | `aurorajammees-growth-ads-operator` | 404 | not possible |
| Senior Project Manager | `senior-project-manager` | 404 | not possible |
| aurorajammee's AI Engineer | `aurorajammees-ai-engineer` | 404 | not possible |
| Bug Analysis Agent | `bug-analysis-agent` | 404 | not possible |
| Podcast Strategist | `podcast-strategist` | 404 | not possible |
| Backend Architect | `backend-architect` | 404 | not possible |
| Pitch Deck Builder | `pitch-deck-builder` | 404 | not possible |
| Search Query Analyst | `search-query-analyst` | 404 | not possible |
| Git Workflow Master | `git-workflow-master` | 404 | **attempted → 404 Not Found** |
| Tracking & Measurement Specialist | `tracking-measurement-specialist` | 404 | not possible |
| Sprint Prioritizer | `sprint-prioritizer` | 404 | not possible |
| Model QA Specialist | `model-qa-specialist` | 404 | not possible |
| Analytics Reporter | `analytics-reporter` | 404 | not possible |
| Social Media Strategist | `social-media-strategist` | 404 | not possible |
| Image Prompt Engineer | `image-prompt-engineer` | 404 | not possible |
| Corporate Training Designer | `corporate-training-designer` | 404 | not possible |
| Superagent — mem20 Capacity Layer | `superagent-mem20-capacity-layer` | 404 | not possible |
| Channel Distribution Agent | `channel-distribution-agent` | 404 | not possible |
| Principal Architect | `principal-architect` | 404 | not possible |
| Short-Video Editing Coach | `short-video-editing-coach` | 404 | not possible |
| Email Marketing Strategist | `email-marketing-strategist` | 404 | not possible |
| GitHub Extensions & Copilot Liaison | `github-extensions-copilot-liaison` | 404 | not possible |
| Minimal Change Engineer | `minimal-change-engineer` | 404 | not possible |
| Autonomous Optimization Architect | `autonomous-optimization-architect` | 404 | not possible |
| Slides Assistant | `slides-assistant` | 404 | not possible |
| Visual Storyteller | `visual-storyteller` | 404 | not possible |
| Identity Graph Operator | `identity-graph-operator` | 404 | not possible |
| Global Podcast Strategist | `global-podcast-strategist` | 404 | not possible |

## Table 2 — handle exists, but belongs to an unrelated third party (24 agents)

`GET https://api.github.com/users/<handle>` → **200**, but the account is a real, unrelated
person or organisation — not the Teamily agent. **These must not be invited.** Adding them
would hand push access to this repository to a stranger.

| Agent name | Derived handle | What the account actually is | Invite |
|---|---|---|---|
| Accounting Assistant | `accounting-assistant` | unrelated user ("accounting assistant") | **do not invite** |
| UX Researcher | `ux-researcher` | unrelated user | **do not invite** |
| Travel Planner | `travel-planner` | unrelated travel app | **do not invite** |
| Code Reviewer | `code-reviewer` | unrelated user | **do not invite** |
| DevOps Automator | `devops-automator` | unrelated user | **do not invite** |
| API Tester | `api-tester` | unrelated user ("api-tester (Snow)") | **do not invite** |
| QA Tester | `qa-tester` | unrelated user | **do not invite** |
| Data Engineer | `data-engineer` | unrelated user | **do not invite** |
| Psychologist | `psychologist` | unrelated user | **do not invite** |
| Super Agent | `super-agent` | unrelated user (4 repos) | **do not invite** |
| Growth Hacker | `growth-hacker` | unrelated user ("Growth Hacker") | **do not invite** |
| Finance Assistant | `finance-assistant` | unrelated user (`finance_assistant`) | **do not invite** |
| Reality Checker | `reality-checker` | unrelated user | **do not invite** |
| Health Assistant | `health-assistant` | unrelated user | **do not invite** |
| Senior Developer | `senior-developer` | unrelated user | **do not invite** |
| Prompt Engineer | `prompt-engineer` | unrelated user ("Prompt-Engineer") | **do not invite** |
| Product Manager | `product-manager` | unrelated user | **do not invite** |
| AI Stock Assistant | `ai-stock-assistant` | unrelated user | **do not invite** |
| Software Architect | `software-architect` | unrelated user | **do not invite** |
| Market Research | `market-research` | unrelated user ("Market-research") | **do not invite** |
| Frontend Developer | `frontend-developer` | unrelated user | **do not invite** |
| Grant Writer | `grant-writer` | unrelated user | **do not invite** |
| Webpage Developer | `webpage-developer` | unrelated user ("Wordpress Developer") | **do not invite** |
| Content Creator | `content-creator` | unrelated organisation | **do not invite** |

## Totals

| Outcome | Count |
|---|---|
| Handle does not exist (404) | 29 |
| Handle exists but is an unrelated third party | 24 |
| **Agents successfully added as collaborators** | **0** |
| **Total AI agents in the group** | **53** |

## What to do instead

Since no agent can hold a GitHub account, attribution is handled in-band:

1. **Section heading** — each contributor's section in `audit.md` starts with
   `## <agent-name>`, exactly as it appears in the group.
2. **Commit trailer** — add `Co-authored-by: <Agent Name> <handle@agents.mem20-audit.local>`
   so the commit history records the author.
3. **Fork + PR** — the repo is public, so any contributor can fork and open a PR against
   `main` with no collaborator invite at all. This is the working path today.

If a real GitHub handle is ever issued for an agent, add it to this table and invite it with
`push` permission.
