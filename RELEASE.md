# Release Cadence

**Organization:** professeer  
**Last updated:** 2026-05-10

---

## Overview

Monthly releases on the **last Sunday of each month**. One milestone per release per repo. Code freeze is 2 weeks before release day — from freeze to release is an all-QA sprint.

| Release | Code freeze | QA window | Prod deploy |
|---------|-------------|-----------|-------------|
| v1.1 – May 2026 | 2026-05-17 (Sun) | 05-17 → 05-31 | 2026-05-31 (Sun) |
| v1.2 – June 2026 | 2026-06-14 (Sun) | 06-14 → 06-28 | 2026-06-28 (Sun) |

---

## Branch Flow

```
feature/<name>  →  qa  →  prod
```

- All feature work branches from `qa` (platform) or `main` (scrapers, research).
- PRs merge back to the same base branch.
- When a release is ready, `qa` is merged to `prod` (platform) or `main` is tagged (scrapers, research).
- `prod` is never committed to directly.

---

## Sprint Calendar (per milestone)

Each milestone spans ~4 weeks, split into two sprints:

| Period | Days | Activity |
|--------|------|----------|
| Sprint A — Development | Days 1–14 | Feature work, bug fixes, PRs merged to `qa` |
| Code freeze | Day 15 (Sunday) | No new features. Only bugfixes and blockers. |
| Sprint B — QA | Days 15–28 | QA testing, bug triage, staging verification |
| Prod deploy | Day 28/29 (Sunday) | Merge `qa → prod`. Tag the release. Close the milestone. |

**Sprint A** goal: all milestone issues merged by freeze date.  
**Sprint B** goal: zero P0/P1 open bugs against this milestone.

---

## Milestones vs. Labels

Milestones are the only release-tracking mechanism. There are no release labels.

- Every milestone issue is assigned a GitHub Milestone (`v1.1 – May 2026`, etc.).
- Priority labels (`P0:critical` → `P4:someday`) reflect urgency, not release membership.
- A milestone is closed only when all its issues are resolved or moved to the next milestone.

---

## Hotfix Protocol

For production breaks discovered after release:

1. Create an issue tagged `P0:critical`.
2. Branch from `prod`: `fix/<short-description>`.
3. PR → `prod` (with Siddarth approval).
4. Cherry-pick or re-merge to `qa` so the fix is in the next release.
5. Tag a patch release (e.g., `v1.1.1`) only if the fix involves a scrapers or data pipeline change that consumers need to track.
6. Post a root-cause comment on the issue after deploying.

---

## Slip Rule

If a milestone cannot ship on its target Sunday:

1. Flag it on the milestone by end of Sprint A.
2. Identify which issues are blockers vs. can-slip.
3. Move non-blockers to the next milestone. Ship what's ready.
4. Do not delay the release date — slip the scope, not the date.

Exception: if >50% of milestone issues are blocked, discuss with Siddarth before proceeding.

---

## Release Checklist

Before deploying to prod on release Sunday:

- [ ] All milestone issues are Closed or moved to next milestone
- [ ] Zero open `P0:critical` or `P1:high` bugs on this milestone
- [ ] Staging (`qa`) has been smoke-tested this week
- [ ] Database migrations (if any) are reviewed and have a rollback plan
- [ ] Scrapers deploy (if changed): verify cron jobs and OCR pipeline are healthy
- [ ] Tag the release in GitHub: `git tag v1.1.0` on prod/main
- [ ] Close the milestone in GitHub
- [ ] Post a one-line deploy note on the relevant GitHub milestone (or a Discussion)
