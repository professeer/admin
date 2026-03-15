# GitHub Project Setup & Management

A team playbook and onboarding guide for the **professeer** GitHub organization.

**Audience:** Siddarth (`reasonablecode`), Bhargavi (`bhargavizaveri`), Gokul (`gokulksunoj`), Srikanth (`srikanthrajkumar`), and anyone joining the team.

**Last updated:** 2026-03-15

---

## Table of Contents

1. [Getting Started](#1-getting-started)
2. [Organization & Repo Structure](#2-organization--repo-structure)
3. [Folder & File Naming Conventions](#3-folder--file-naming-conventions)
4. [Branching & Code Review](#4-branching--code-review)
5. [Project Management Workflow](#5-project-management-workflow)
6. [Issue Discipline & Communication](#6-issue-discipline--communication)
7. [Best Practices](#7-best-practices)

---

## 1. Getting Started

### Install and authenticate `gh` CLI

The `gh` CLI is our primary tool for interacting with GitHub from the command line. All team members should have it set up.

**Install:**

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Or download from https://cli.github.com/
```

**Authenticate:**

```bash
gh auth login
# Choose: GitHub.com → HTTPS → Login with web browser

# Verify
gh auth status
# Should show: Logged in to github.com as YOUR_USERNAME
```

**Set default org (optional, saves typing):**

```bash
gh config set default_owner professeer
```

**Quick sanity check:**

```bash
# List repos you have access to
gh repo list professeer

# List open issues on platform
gh issue list -R professeer/platform
```

### SSH key setup

If you don't already have an SSH key linked to GitHub:

```bash
# Generate a key
ssh-keygen -t ed25519 -C "your-email@example.com"

# Add to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub
# Paste at: https://github.com/settings/ssh/new

# Test
ssh -T git@github.com
```

### Clone the repos

```bash
git clone git@github.com:professeer/platform.git
git clone git@github.com:professeer/scrapers.git
git clone git@github.com:professeer/research.git
git clone git@github.com:professeer/admin.git
```

---

## 2. Organization & Repo Structure

### Repos

| Repo | Purpose | Default Branch |
|------|---------|----------------|
| `professeer/platform` | Frontend (React) + backend (FastAPI), product features, data quality | `qa` |
| `professeer/scrapers` | Court scrapers, OCR pipeline, data pipeline | `main` |
| `professeer/research` | Notebooks, analysis, papers | `main` |
| `professeer/admin` | Org-level docs, templates, processes (this document lives here) | `main` |

### Team Roles

| GitHub handle | Name | Role |
|---------------|------|------|
| `reasonablecode` | Siddarth | Engineering lead. Approves PRs on platform and scrapers. |
| `bhargavizaveri` | Bhargavi | CEO. Owns product priorities, milestone planning, feature prioritization. |
| `gokulksunoj` | Gokul | Feature extraction, research. |
| `srikanthrajkumar` | Srikanth | Scrapers, QA/QC. |

All team members have push access to all repos.

---

## 3. Folder & File Naming Conventions

### Folder names

- **Always lowercase.** No spaces, no camelCase.
- Use hyphens to separate words: `data-pipeline/`, not `DataPipeline/` or `data_pipeline/`.
- Keep folder structures shallow and readable. If someone can't find a file in 3 clicks, restructure.

### File naming

For **code files**, follow the language convention (e.g., `snake_case.py`, `kebab-case.tsx`).

For **documents, reports, and plans**, use this naming pattern:

```
yyyymmdd_filename_project_version.ext
```

Examples:
- `20260315_scraper_audit_bomhc_v1.md`
- `20260301_sprint_plan_milestone2_v2.md`
- `20260220_api_design_courtsdaily_v1.pdf`

**Why this format:**
- Date prefix makes files sortable chronologically
- Project/context tag makes files searchable
- Version suffix avoids duplicate filenames like `plan_final_FINAL_v2.md`

### Version control for documents

- **If a document lives in a git repo** (which it should), git history IS your version control. You don't need `_v1`, `_v2` suffixes — just commit with a clear message.
- **Use version suffixes only** when sharing documents outside git (email attachments, Google Drive, etc.) or when the document represents a fundamentally different version (e.g., a rewrite, not an edit).
- **Never keep multiple copies** of the same document in the same folder. If you need the old version, it's in git history.

---

## 4. Branching & Code Review

### When to branch

| Change size | Workflow |
|-------------|----------|
| **Small fix** (typo, one-line bug, config tweak) | Commit directly to the working branch. No PR needed. |
| **Medium change** (new feature, refactor, multi-file edit) | Create a branch, commit regularly, open a PR. |
| **Large change** (new system, architectural change) | Create a branch, break work into small commits, open a PR. Consider stacking PRs if the diff exceeds ~400 lines. |

### Branch naming

```
<type>/<short-description>
```

Examples:
- `feature/delhi-hc-scraper`
- `fix/search-pagination`
- `docs/api-reference`

### Commit messages

Use imperative mood. Reference issue numbers.

```
Add Delhi HC scraper support

Implements case listing and detail scraping for Delhi High Court.
Handles captcha solving via Gemini fallback.

Fixes #42
```

**Rules:**
- First line: imperative verb, under 72 characters
- Blank line, then body if needed
- Reference issues: `Fixes #42`, `Closes #42`, `Relates to #42`

### Pull requests

- **Title:** Short, under 70 characters.
- **Body:** What it does, why, how to test.
- **Link the issue:** Use `Closes #N` in the body so the issue auto-closes on merge.
- **Keep PRs focused.** One PR = one logical change. If you're touching 3 unrelated things, that's 3 PRs.
- **Target size:** Under ~400 lines of diff. If larger, consider splitting.

### Code review

- **PRs on `platform` and `scrapers` require approval from Siddarth** (`reasonablecode`) before merging.
- **Review is a mix of AI and human evaluation.** PRs may be reviewed by automated tools, AI assistants, or team members. Both types of feedback are valid.
- **Review SLA:** Respond to review requests within 1 business day.
- **When reviewing:** Focus on correctness, readability, and whether the change does what the issue asks. Nitpicks are fine but label them as such.

### Merging

- Prefer **squash merge** for feature branches (keeps history clean).
- Use **merge commit** for long-lived branches or when individual commits matter.
- Delete the branch after merging.

---

## 5. Project Management Workflow

### Cadence

| Unit | Duration | Purpose |
|------|----------|---------|
| **Sprint** | 15 days | A work cycle. Mix of tasks decided during sprint planning. |
| **Milestone** | 2+ sprints (30+ days) | A goal-based deliverable. Done when its issues are closed. |

### Sprint planning

At the start of each sprint:

1. Review the project board — what's in progress, what's blocked, what's done.
2. Pull issues from the milestone backlog into the sprint.
3. Assign owners. Mix of types: features, bugs, research, QA.
4. Set deadlines on sprint issues.
5. Record the sprint plan as a GitHub issue or discussion.

### Milestone planning

- **Bhargavi** owns milestone goals, feature prioritization, and product direction.
- Milestones are created in GitHub: `Settings → Milestones → New milestone`.
- Each milestone has a clear goal statement and a target date.
- Everyone contributes to milestone planning, but Bhargavi holds the final decision on product scope.

### Issue lifecycle

```
Created → Inbox → Triaged → In Progress → In Review → Done
```

**Project board fields** (set on every issue):

| Field | Options |
|-------|---------|
| **Status** | Inbox, Triaged, In Progress, In Review, Done |
| **Priority** | P0:critical, P1:high, P2:medium, P3:low, P4:someday |
| **Effort** | XS (< 1 day), S (2-3 days), M (1 week), L (2 weeks), XL (1 month) |
| **Court** | BHC, DHC, NCLT, SC, All |
| **Vertical** | Scraper, Product & Frontend, Feature Extraction, Research & Writing, Business & Admin, QA/QC |

### Labels

Every issue gets **one type label** and **one priority label**:

**Type labels:** `bug`, `feature`, `enhancement`, `research`, `qa-qc`, `infra`, `docs`, `business`

**Priority labels:** `P0:critical`, `P1:high`, `P2:medium`, `P3:low`, `P4:someday`

**Quality label:** `needs-definition` — applied when the issue lacks clear scope or acceptance criteria.

### Issue assignment

- **Anyone can assign issues to anyone**, including themselves.
- **Anyone can create issues.** If you see a bug, create an issue and fix it. Even if it's small. The record helps against future reported bugs.
- **Small issues don't need permission.** If it's not on the milestone and it's small, just do it and document it.
- **Milestone issues** are planned during sprint planning.

---

## 6. Issue Discipline & Communication

### Issue creation checklist

Before creating an issue, make sure it has:

- [ ] **Clear title** — verb-first, specific (e.g., "Add Delhi HC scraper support", not "Delhi HC")
- [ ] **Problem statement** — 1-2 sentences: what's wrong, what's missing, or what's needed
- [ ] **Acceptance criteria** — checkbox list of specific, verifiable outcomes
- [ ] **Context** — links, screenshots, related issues, who requested it
- [ ] **One type label** — `bug`, `feature`, `enhancement`, `research`, `qa-qc`, `infra`, `docs`, or `business`
- [ ] **One priority label** — `P0:critical`, `P1:high`, `P2:medium`, `P3:low`, or `P4:someday`
- [ ] **Assignee** — who's responsible (can be yourself)
- [ ] **Milestone** — if it belongs to a current milestone
- [ ] **Added to project board** — with Status, Priority, Effort, Court, and Vertical fields set

If an issue is missing scope or acceptance criteria, apply the `needs-definition` label and add an "Open Questions" section listing what needs to be answered before work can start.

### Issue body template

```markdown
## Problem / Context

[1-2 sentences: what's wrong, what's missing, or what's needed]
[Include: who requested it, relevant links, screenshots]

## Acceptance Criteria

- [ ] [Specific, verifiable outcome]
- [ ] [Another outcome]
- [ ] [How to verify it's done]

## Notes

[Optional: technical approach, constraints, dependencies, related issues]
```

### Regular updates — non-negotiable

**If you need to be asked for a status update, it's a failure.**

- Update your issues regularly. At minimum, comment when:
  - You start working on it
  - You hit a blocker
  - You make significant progress
  - You finish it
- If an issue is "In Progress" for more than 3 days with no update, that's a problem.

### Deadline management

- Every sprint issue should have a deadline.
- **2 days before deadline:** If you think you might miss it, say so. Flag it on the issue. Propose a new date.
- **1 day before deadline:** This is the absolute worst acceptable time to raise a flag. By this point you should already have escalated.
- **After deadline with no update:** This should never happen.

The point is not to punish missed deadlines — it's to make them visible early so the team can adjust.

### Communication SLA

| Channel | Response time |
|---------|---------------|
| **GitHub comments** (issues, PRs, reviews) | < 6 hours on weekdays |
| **P0:critical issues** | Immediate (WhatsApp/call + GitHub) |
| **Weekend** | Best effort. No SLA enforced. |

### Where communication happens

- **WhatsApp, email, Google Chat** — fine for quick questions, discussions, coordination.
- **GitHub** — the system of record. If a decision was made on WhatsApp, capture it on the relevant GitHub issue. If it's not on GitHub, it didn't happen (for record-keeping purposes).

---

## 7. Best Practices

### Definition of Done

An issue is "Done" when:

- [ ] Code is merged to the target branch
- [ ] Tests pass (if applicable)
- [ ] Documentation updated (if the change affects how something works)
- [ ] The issue's acceptance criteria are all checked off
- [ ] The issue is moved to "Done" on the project board

### Stale issue hygiene

- Issues untouched for 30+ days get reviewed during sprint planning.
- If still relevant: re-prioritize and assign.
- If not relevant: close with a comment explaining why.
- Don't let the backlog become a graveyard.

### PR size guidelines

- Keep PRs under ~400 lines of diff.
- If a feature is large, break it into stacked PRs or sequential PRs that each deliver a coherent piece.
- Smaller PRs get reviewed faster and have fewer bugs.

### Secrets and sensitive data

- **Never commit** `.env` files, API keys, passwords, or credentials.
- Use `.env.example` files to document what environment variables are needed (with placeholder values).
- If you accidentally commit a secret, rotate it immediately. Git history is permanent.
- `.gitignore` should include: `.env`, `.env.*`, `!.env.example`, `credentials/`, `*.pem`, `*.key`.

### Incident response (P0 flow)

When something is broken in production:

1. **Create an issue immediately** — tag `P0:critical`, assign yourself.
2. **Notify on WhatsApp** — don't assume people will check GitHub.
3. **Fix takes priority** over all sprint work.
4. **Post a root cause comment** on the issue after the fix. What broke, why, how to prevent it.

### Retrospectives

At the end of each milestone, do a brief async retro:

- Create a GitHub discussion or issue titled `Retro: [Milestone Name]`
- Each team member adds:
  - **What worked** — keep doing this
  - **What didn't** — stop or change this
  - **One thing to try next milestone**
- Review together and pick 1-2 concrete changes for the next milestone.

### Commit hygiene

- Commit early and often. Small, focused commits are easier to review and revert.
- Don't commit generated files, build artifacts, or IDE configs.
- Don't commit `node_modules/`, `__pycache__/`, `.venv/`, or `data/` directories.

### Quick reference: common `gh` commands

```bash
# Issues
gh issue create -R professeer/platform --title "Title" --body "Body" --label "bug,P1:high"
gh issue list -R professeer/platform --state open
gh issue close 42 -R professeer/platform
gh issue comment 42 -R professeer/platform --body "Fixed in abc123"
gh issue edit 42 -R professeer/platform --add-label "P0:critical"

# Pull Requests
gh pr create -R professeer/platform --title "Title" --body "Closes #42"
gh pr list -R professeer/platform
gh pr merge 7 -R professeer/platform --squash

# Project Board
gh project item-list 1 --owner professeer
gh project item-add 1 --owner professeer --url <issue-url>

# Repo
gh repo list professeer
```

---

## Appendix: Useful Links

- **Project board:** https://github.com/orgs/professeer/projects/1
- **Platform repo:** https://github.com/professeer/platform
- **Scrapers repo:** https://github.com/professeer/scrapers
- **Research repo:** https://github.com/professeer/research
- **Admin repo:** https://github.com/professeer/admin
