# Professeer Development Process

**Status:** Draft v1 for adoption — supersedes `RELEASE.md` §Branch Flow and extends `github-project-setup-management.md`
**Applies to:** Siddarth (`reasonablecode`), Srikanth (`srikanthrajkumar`), Gokul (`gokulatwork26`), Bhargavi (`bhargavizaveri`), and all AI agents working in the org
**Last updated:** 2026-06-11

---

## TL;DR — the eight rules

1. **Every change starts with an issue.** No issue, no branch.
2. **Branches are named for their issue**: `feat/123-short-name`, `fix/51-short-name`.
3. **All code reaches `qa` through a squash-merged PR** whose body says `Closes #123`. Never push to `qa` or `prod` directly.
4. **Code reaches `prod` only via the monthly promotion PR** (`qa → prod`), or a hotfix PR.
5. **Servers are pull-only.** Nobody commits, edits, or branches on qa-prof, valaritas, or coppermind. Development happens on dev boxes (waystone, Srikanth's box) or laptops.
6. **Monday 30-minute triage** is the weekly sprint plan. The project board is the plan — no separate documents.
7. **Monthly cadence:** build weeks 1–2 → freeze middle Sunday → QA weeks 3–4 → ship last Sunday.
8. **Humans log in as themselves; only automation logs in as `deploy`.** Becoming deploy is `sudo -iu deploy` from your own account — logged with your name, every time. Dev boxes get read-only reach into QA/prod, never shells (§1, "Server login model").

We are on the free GitHub plan, so rules 1–4 are honor code. Everything else is mechanical: only Siddarth and Srikanth hold write access on code repos (repo roles, §1), and servers physically cannot push (read-only deploy keys + deploy-script gates, §9). The Friday hygiene check (§10) catches honor-code drift weekly so it never compounds.

---

## 1. People, environments, and what's allowed where

### People

| Person | Role | Reviews |
|---|---|---|
| Siddarth | Eng lead — platform, infra, releases | Reviews all of Srikanth's PRs; self-merges own `qa` PRs after AI review |
| Srikanth | Scrapers, QA/QC | Smoke-test sign-off on every prod promotion |
| Gokul | Feature extraction, research | — |
| Bhargavi | CEO — priorities, milestones | Milestone scope decisions |

### Environments

| Box | Role | Checked-out branch | Writes allowed? |
|---|---|---|---|
| waystone (`vm-dev-siddarth`) | Siddarth dev | feature branches | **Yes — this is where code is written** |
| `vm-dev-srikanth` | Srikanth dev | feature branches | **Yes** |
| qa-prof (fishery) | QA / staging | `qa` only | No. Pull-only. |
| professeer (valaritas) | Production — platform | `prod` only | No. Pull-only. |
| coppermind | Production — scrapers | `prod` only | No. Pull-only. |

**Pull-only means:** the `deploy` user on QA/prod boxes only ever runs `deploy.sh` (§9). No editors on tracked files, no `git commit`, no worktrees, no local branches. If you find yourself wanting to edit a file on a server, that's an issue + a branch + a PR — the deploy script will refuse a dirty tree anyway.

**Mechanical backstop:** QA/prod servers authenticate to GitHub with **read-only deploy keys** (free on any plan). Even a moment of weakness can't push from a server.

The one exception to pull-only: genuine server *configuration* (nginx, cron, docker daemon) that lives outside the repos. Document such changes in `server-setup` within the same day.

### Access model — who can write what

Three layers of isolation, two of them mechanical:

| Layer | Rule | Enforcement |
|---|---|---|
| **Who can push to a repo** | Code writers are **Siddarth and Srikanth only**. Gokul and Bhargavi get the **Triage** role on `platform` and `scrapers` — full issue/PR/board management, comments, labels; no push. Gokul keeps Write on `research`. | **Mechanical** — repo roles, free on any plan |
| **Which branches a writer touches** | Writers push feature branches only; `qa` and `prod` change via PRs | Honor code + Friday hygiene report (per-branch restriction needs a paid plan) |
| **What servers can do** | `deploy` user pulls and executes — nothing else | **Mechanical** — read-only deploy keys + `deploy.sh` gates |

Known exception: Bhargavi and Siddarth are org owners, and owners always retain admin on every repo. That's deliberate (continuity if Siddarth is unavailable), not a workflow path — Bhargavi's work lives in issues and milestones, not pushes. If Gokul later needs to land code (e.g., feature-extraction work in `platform`), bump him to Write at that point and fold him into §4–5; don't pre-grant.

### Server login model — who may SSH as whom

Same philosophy as repo access: humans act under their own names, service accounts do exactly one job, and everything that can be mechanical is.

| Principle | Rule | Enforcement |
|---|---|---|
| **One human = one account** | Siddarth and Srikanth each have a personal account on every box they touch. Nobody SSHes in as `deploy`. | **Mechanical** — `deploy`'s authorized_keys holds automation keys only |
| **`deploy` is a service account** | Every key in its authorized_keys belongs to a machine, not a person, and is pinned to a forced command (`command="..."`) wherever the use-case allows. Its password stays **locked**, so raw `su deploy` is impossible by construction — there is no unlogged path into the account. | **Mechanical** — forced-command keys + locked password |
| **Escalation is explicit and attributed** | Becoming deploy = `sudo -iu deploy` from your personal account. Root work = `sudo` from your personal account, password required. Every invocation lands in `/var/log/auth.log` with the real username, TTY, and command. | **Mechanical** — PAM/rsyslog defaults; retention bumped in §11 |
| **Dev boxes read, never write** | waystone reaches the servers only through forced-command keys: `rrsync -ro /` for files, plus `permitopen`-scoped tunnels to local DB ports. A dev box must never hold a key that yields a *shell or exec* on QA/prod — not as deploy, and not as a personal user either (shell + sudo = deploy, defeating the point). Interactive shells on servers come from laptops only. `restrict` alone is NOT enough: it blocks the PTY but still allows command execution — the `command="..."` clause is what actually closes exec. | **Mechanical** — `restrict,command=...` in authorized_keys |
| **No agent forwarding** | Never `ssh -A` into waystone or any server. A forwarded agent lets the box borrow your laptop key — and the laptop key opens personal accounts (and today, deploy) on QA/prod. | Honor code — keep `ForwardAgent no` (the default) |
| **Per-box user whitelist** | `AllowUsers` in sshd_config names exactly the expected accounts; `PermitRootLogin no`; `PasswordAuthentication no`. Accounts of departed collaborators are removed the week they leave, not parked. | **Mechanical** — sshd config |
| **waystone is single-user** | Only `siddarth` exists (stock `ubuntu` account removed). On GCP the real gate is IAM: anyone who can edit instance metadata or open IAP tunnels can mint users — so "one user" is ultimately project IAM membership, backed by `block-project-ssh-keys=TRUE` so project-wide keys can't silently add accounts. | **Mechanical** — GCP IAM + instance metadata |

Steady state, per box:

| Box | Personal accounts | `deploy` authorized_keys may contain | Humans become deploy via |
|---|---|---|---|
| waystone | siddarth only | n/a — no deploy user | n/a |
| vm-dev-srikanth | srikanth only | n/a — no deploy user | n/a |
| qa-prof (fishery) | siddarth, srikanth | automation only: waystone `rrsync -ro` + tunnel key, cross-server sync keys (each forced-command) | `sudo -iu deploy` |
| valaritas | siddarth, srikanth | automation only, same rule | `sudo -iu deploy` |
| coppermind | siddarth, srikanth (when needed) | automation only, same rule | `sudo -iu deploy` |

vm-dev-srikanth is Srikanth's to provision, on the waystone template: single personal user, `rrsync -ro` forced-command keys for read access to fishery/valaritas (with `command=` on **every** box that trusts the key — see the coppermind lesson, §11 #14), and no shell-capable key to any server.

**su/sudo logging.** Ubuntu already logs every `su` and `sudo` invocation to `/var/log/auth.log` with the invoking user — the audit trail is mechanical and free. What is *not* default: retention (4 weeks → 12 months, §11) and tamper-resistance (anyone with root can edit local logs). For a two-admin team the local trail is proportionate; when a third person gets server access, forward `auth.*` to a second box so no single root can scrub their own tracks.

**End state — humans rarely become deploy at all.** Routine deploys move to GitHub Actions (§9 Stage 1): the Action gets its own SSH key in deploy's authorized_keys, pinned with `command="/opt/deploy.sh"` like every other automation key — a leaked Actions secret can trigger a deploy, never open a shell. Once the button exists, `sudo -iu deploy` is reserved for hands-on debugging: the logged exception, not the routine.

**Why deploy keeps docker but loses sudo:** `deploy.sh` needs git and docker (group membership) — it does not need root. Once personal-account sudo is verified on a box, `deploy` leaves the sudo group and `/etc/sudoers.d/deploy` goes away (§11): the account every automation key can reach should not be a free root.

**What this guards, honestly:** attribution and accident-prevention — a mistyped command, a confused AI agent, a borrowed laptop key. It does not stop a malicious admin: docker-group membership is root-equivalent, and both admins hold sudo. That is the right tradeoff at this team size; revisit alongside the GitHub Team upgrade, which fixes the repo-side honor code but changes nothing here — these guards are independent of the GitHub plan.

---

## 2. Branch model

Same model for every deployable repo. No exceptions, no per-repo variants.

```
feat/123-thing ──squash PR──▶ qa ──monthly promotion PR──▶ prod
fix/51-thing  ──squash PR──▶ qa
fix/200-hotfix ──hotfix PR──▶ prod ──cherry-pick──▶ qa
```

| Repo | Long-lived branches | Notes |
|---|---|---|
| `platform` | `qa` (default), `prod` | Delete stale `main` |
| `scrapers` | `qa` (default), `prod` | Rename `main` → `qa` during adoption (§11) |
| `research` | `main` | Lightweight: direct commits OK, cite issue in message |
| `admin` | `main` | Lightweight: direct commits OK |
| `courts-daily-usage-report` | `main` | Lightweight: direct commits OK, but file issues for non-trivial work |

There is **no `dev` branch**. "Dev" is a place (your dev box), not a branch. Feature branches off `qa` serve that purpose.

**Branch naming:** `<type>/<issue-number>-<short-kebab-description>` where type ∈ `feat`, `fix`, `chore`, `data`, `docs`. Examples from our own history that got this right: `fix/dhc-disposal-date-parser-issue-51` (works, but put the number after the slash: `fix/51-dhc-disposal-date-parser`).

**Branch lifetime:** branches live days, not weeks. Merged branches are deleted at merge time (enable "Automatically delete head branches" in repo settings — free). A branch older than one sprint is a smell; split the issue.

---

## 3. Issues — what needs one, and what a good one looks like

### When an issue is required

- Any change that alters behavior on `qa` or `prod` — features, fixes, data corrections, migrations, scraper changes.
- Any bug found during QA weeks. One issue per discrete defect.
- Any task you expect to take more than ~30 minutes.

### When an issue is NOT required

- Typos, comment fixes, plan documents, research notebooks.
- Small chores can be batched: each milestone may carry one `chore: <milestone> maintenance` issue that collects them, instead of forcing ceremony.

### Quality bar

Per the issue-manager conventions (which we'll keep): verb-first title, problem statement, acceptance criteria as checkboxes, one type label + one priority label, a milestone, an assignee. Issues that don't meet the bar get `needs-definition` with open questions listed.

### Banned: bucket issues

No more rolling mega-threads ("NCLT Bugs", 26 comments). A bucket issue is where traceability goes to die — work hides in comments, nothing links to commits, nothing ever closes. If a theme is real, make it a **milestone or a label**, and file each defect as its own issue. Existing buckets get decomposed and closed during adoption (§11).

### Discussion lives on issues

Design debates, QA findings, decisions — comment on the relevant issue, not in chat that evaporates. If a discussion produced a decision, the decision gets one summary comment. This is the fix for "lots of comments are not against issues": the comment's home is wherever `Closes #N` will eventually point.

---

## 4. The development loop

This is the daily workflow. The discipline lives at the **PR level**, not the commit level — individual commits on your branch can be messy; nobody will ever see them after the squash.

```bash
# 1. Pick an issue (it's already triaged, labeled, milestoned — see §6)
gh issue view 123

# 2. Branch from fresh qa
git fetch origin && git switch -c feat/123-dhc-enrichment origin/qa

# 3. Work. Commit as often as you like, messages need not be ceremonial.

# 4. Open the PR — title in conventional-commit form (it becomes the qa commit)
gh pr create --base qa \
  --title "feat(dhc): enrich case metadata from cause lists" \
  --body "Closes #123

## What
...
## How verified
..."

# 5. Merge (squash — the only merge method enabled on the repo)
gh pr merge --squash --delete-branch
```

**Why squash-only:** every commit landing on `qa` automatically carries its PR number; the PR carries `Closes #123`; the issue closes itself on merge. Issue ↔ PR ↔ commit linkage becomes a byproduct of the workflow instead of a per-commit tax — this is the part of the old process that was too expensive to follow, made nearly free. Set squash as the *only* allowed merge method in repo settings (free plan supports this), with "default to PR title" for the commit message.

**PR body must contain `Closes #N`** (or `Fixes #N`). The CI check (§10) flags PRs that don't.

**AI agents** (Claude, Codex) follow the same loop: they work on issue-named branches, and their PRs carry `Closes #N` like anyone else's. No more commits authored directly to `qa` by an agent.

---

## 5. Review rules — sized for a 1.5-person team

Requiring human review on every PR would deadlock a team where one person writes 95% of the code. So:

| PR | Gate before merge |
|---|---|
| Srikanth → `qa` | Siddarth reviews and approves |
| Siddarth → `qa` | AI review (Codex/Claude) + self-merge. No human approval needed. |
| Anyone → `prod` (promotion or hotfix) | Promotion checklist (§7) + Srikanth smoke-test sign-off comment |

Review SLA: 24 hours on weekdays. If a PR waits longer, ping on the PR — and if it's blocking, the author may escalate to a direct message.

---

## 6. Cadence

### Weekly: Monday triage (30 minutes, Siddarth + Srikanth; Bhargavi optional)

This *is* the weekly sprint plan. Standing agenda:

1. **Empty the Inbox** on the project board: every new issue gets type + priority labels, a milestone (or `P4:someday`), and an assignee or back to backlog.
2. **Pick the week:** each person moves their 3–5 issues for the week to **In Progress** on the board. The board is the plan — no separate sprint documents.
3. **Check milestone burn-down:** is the release milestone on track for freeze? If not, apply the slip rule *now* (slip scope, never the date).
4. **Review the Friday hygiene report** (§10) — call out any honor-code drift.

### Monthly: the release calendar (unchanged from RELEASE.md)

One milestone per release per repo. Release = milestone = last Sunday of the month.

| Period | Activity |
|---|---|
| Days 1–14 (weeks 1–2) | Build. Feature PRs merge to `qa` continuously. |
| **Middle Sunday — code freeze** | All milestone feature PRs merged to `qa`. After this: bugfixes only. |
| Days 15–28 (weeks 3–4) | QA on qa-prof. Bugs found → issues on the milestone → `fix/` PRs to `qa`. |
| **Last Sunday — release** | Promotion PR `qa → prod` (§7). Deploy. Tag. Close milestone. |

Upcoming: **v1.2 freezes 2026-06-14 and ships 2026-06-28. v1.3 freezes 2026-07-12 and ships 2026-07-26.**

Milestones are the only release-tracking mechanism (no release labels). A milestone closes only when every issue is closed or explicitly moved to the next one — and moving an issue requires a one-line comment saying why.

---

## 7. Releasing: the promotion PR

On release Sunday (or the Friday before, to leave slack):

1. Open a PR from `qa` to `prod` titled `release: v1.2 — June 2026`.
2. The body uses the promotion-spec format we already do well (see platform PR #326): scope table of issues shipped, anything deliberately held back, migration notes with rollback plan, deploy steps.
3. **Checklist in the PR body:**
   - [ ] All milestone issues closed or moved (with reasons)
   - [ ] Zero open P0/P1 bugs on the milestone
   - [ ] qa-prof smoke-tested this week — Srikanth signs off with a comment
   - [ ] DB migrations reviewed + rollback noted
   - [ ] Scrapers (if changed): cron jobs and OCR pipeline verified healthy
4. Merge (regular merge for promotions, not squash — preserve the qa history), then deploy (§9), then `git tag v1.2.0` on `prod`, close the milestone, one-line deploy note on the milestone.

If `qa` contains something that must NOT ship, that's a planning failure to fix in triage — not a cherry-pick adventure on release day. Cherry-pick releases (like the NCLT release branch in June) are the documented exception, not the norm: they require their own spec doc, as PR #326 did.

---

## 8. Hotfixes

For prod breakage that can't wait for the next release:

1. File the issue, label `P0:critical`.
2. Branch **from `prod`**: `fix/200-short-name`.
3. PR to `prod` with `Closes #200`. AI review minimum; human review if time allows.
4. Deploy via `deploy.sh`, verify, then **cherry-pick to `qa`** so the fix rides the next release too.
5. Tag a patch (`v1.2.1`) if consumers need to track it.
6. Root-cause comment on the issue within 24h of the deploy.

---

## 9. Deploys

All deploys are `deploy.sh` on the target box — never a bare `git pull`. The script is our free, mechanical enforcement point:

```
deploy.sh contract (per box):
  1. Abort if working tree is dirty            ← makes server-side edits self-defeating
  2. Abort if checked-out branch ≠ designated  (qa-prof: qa; valaritas/coppermind: prod)
  3. git fetch + fast-forward-only pull        ← refuses diverged history
  4. docker compose up -d --build (per-repo specifics)
  5. Append timestamp + old→new commit to /opt/deploy.log
```

| Box | Deploys | When |
|---|---|---|
| qa-prof | `qa` after merges | Whenever a PR merges to `qa` (deployer = PR author). During QA weeks, at least daily. |
| valaritas | `prod` | Release Sunday + hotfixes only |
| coppermind | `prod` | Release Sunday + hotfixes only. Mind long-running celery jobs: drain/restart workers per runbook. |

Anything a cron job runs **must be a tracked file** in the repo at a tagged commit — never an untracked script (this is how coppermind's nightly run drifted off-book).

### Road to CI/CD — three stages, each earned by the previous one being boring

| Stage | What | Trigger | Prerequisite |
|---|---|---|---|
| **0 — now** | `deploy.sh` run manually over SSH | Human | Adoption checklist done |
| **1** | GitHub Action runs `deploy.sh` over SSH (deploy key + host secrets); deploy log doubles as Actions history | `workflow_dispatch` button — prod deploys become one click with an audit trail | Stage 0 boring for a full release cycle |
| **2** | Auto-deploy `qa` → qa-prof on every push to `qa`; prod stays on the button | Push to `qa` | Stage 1 reliable + CI tests green as a habit |

Prod deploys stay human-triggered indefinitely — the button is the feature, not a stopgap. Full auto-deploy-to-prod only becomes sensible with real branch protection and a trustworthy test suite (i.e., the Team-plan revisit in §10).

Stage 1's SSH credential follows the §1 login model: its key lands in deploy's authorized_keys pinned to `command="/opt/deploy.sh"`, so the GitHub secret can deploy and do nothing else. Stage 1 is also what retires routine `sudo -iu deploy` — once deploys are a button, humans become deploy only to debug.

---

## 10. Keeping the honor code honest

No branch protection on the free plan, so visibility substitutes for enforcement:

1. **CI link check (free Actions):** a tiny workflow on every PR that fails with a red X if the body lacks `Closes #N` / `Fixes #N`. Not blocking — but a red X on your own PR is a nudge that works.
2. **CI tests:** resurrect `test.yml.disabled`, pointed at `qa`/`prod` (it currently targets branches that don't exist). Red X on broken tests, even if merging isn't blocked.
3. **Friday hygiene report (automated, scheduled Action or fleet script), posted where Monday triage reads it:**
   - direct pushes to `qa`/`prod` this week (commits with no associated PR)
   - PRs merged without `Closes #N`
   - open issues missing labels/milestone
   - per-server: dirty tree? unpushed commits? wrong branch? last-deploy age
4. **The standing deal:** anyone may break a rule in a genuine emergency — and then files the issue and a one-line "broke rule X because Y" comment the same day. The honor code's failure mode isn't the exception; it's the *silent* exception.

If three consecutive hygiene reports are clean, nothing changes — we're just disciplined. If they're consistently dirty, the honor-code experiment has its answer and we revisit GitHub Team ($16/mo) for real branch protection.

---

## 11. Adoption checklist (one-time, ordered)

Order matters: GitHub must become the source of truth *before* the rules take effect, or the rules lock the real code out.

1. **coppermind:** push the 40 unpushed `main` commits; commit-or-discard the dirty tree; get `scripts/run_nclt_daily.sh` into git; remove dev worktrees; check out `prod`.
2. **valaritas:** push the 2 unpushed `prod` commits (feedback features); reconcile modified source files; set upstream tracking; switch scrapers remote HTTPS → SSH.
3. **qa-prof:** push the stray `qa` commit; reconcile dirty dbsync scripts.
4. **scrapers repo:** fast-forward `prod` to `main` (it's 7 commits behind, including fixes to its own consolidation merge); rename `main` → `qa`; set `qa` as default; update every clone.
5. **platform repo:** delete dead `main` + merged/stale branches; close finished v1.1 milestone.
6. **Repo settings (all deployable repos):** squash-only merges, default-to-PR-title, auto-delete head branches.
7. **Repo roles:** downgrade Gokul to Triage on `platform` and `scrapers` (currently Write everywhere); keep his Write on `research`. Normalize Srikanth to Write on `courts-daily-usage-report` (currently Admin).
8. **Deploy keys:** replace server credentials with read-only deploy keys on qa-prof, valaritas, coppermind.
9. **`deploy.sh`** written, tested, and committed for all three boxes.
10. **CI:** add the PR link-check workflow; re-enable the test workflow against `qa`/`prod`.
11. **waystone / dev boxes:** configure git identities (waystone currently has none).
12. **Issues:** decompose and close bucket issues ("NCLT Bugs", "Bom HC bugs", "Speed and Latency Bugs", "CourtsDaily | CheckList"); label the unlabeled recent issues; empty the 34-item Inbox in the first Monday triage.
13. **Commit this document** to `professeer/admin`, update `RELEASE.md` to point here, and open an adoption issue assigned to all four team members — acceptance criterion: each person comments that they've read it.
14. **Close the coppermind exec hole (do this first — verified open 2026-06-11):** the `dev-tunnel-waystone-RO` key in `deploy@coppermind`'s authorized_keys has `restrict,port-forwarding,permitopen=...` but no `command=`, so waystone can execute arbitrary commands as deploy on prod (`ssh coppermind id` works today). Prepend `command="/usr/bin/false",` to that line — `ssh -N` tunnels keep working, exec dies. One-line edit, zero downtime.
15. **Personal accounts on fishery and valaritas — in this order, each step verified before the next (lockout safety; Hetzner rescue console is the recovery path):**
    ```bash
    # On the box, from an existing deploy session:
    sudo adduser --gecos "Siddarth Raman" siddarth        # set a strong password — sudo will require it
    sudo usermod -aG sudo siddarth
    sudo install -d -m 700 -o siddarth -g siddarth /home/siddarth/.ssh
    echo "<Mac public key>" | sudo tee /home/siddarth/.ssh/authorized_keys
    sudo chown siddarth:siddarth /home/siddarth/.ssh/authorized_keys && sudo chmod 600 /home/siddarth/.ssh/authorized_keys
    # sshd_config — fishery: AllowUsers deploy srikanth siddarth (atibhi dropped, see #16)
    #               valaritas: add the line (currently has no AllowUsers at all)
    sudo sshd -t && sudo systemctl reload ssh
    # From the Mac, in a NEW terminal (keep the old session open):
    #   ssh siddarth@<box>  →  sudo -iu deploy  →  cd <repo> && git status   — verify both hops
    # Mac ~/.ssh/config: User deploy → User siddarth for "fishery qa-prof" and "valaritas professeer"
    # Only after that works: strip HUMAN keys from /home/deploy/.ssh/authorized_keys
    #   fishery: siddarth@reasonablesolutions.in (Mac RSA) + srikanthrajkumar26@gmail.com
    #   valaritas: siddarth@reasonablesolutions.in (Mac RSA)
    # One week later, deploy.sh + cron proven healthy under the new flow, demote the service account:
    #   sudo deluser deploy sudo && sudo rm /etc/sudoers.d/deploy    # deploy keeps the docker group
    ```
    The waystone `rrsync -ro` keys are NOT touched by any of this — read-only dev access keeps working throughout. Note `su deploy` cannot work (deploy's password is locked — deliberately); `sudo -iu deploy` is the only road, which is exactly what makes every crossing logged.
16. **Stale and lateral access:** remove `atibhi` from fishery (`sudo deluser --remove-home atibhi` + drop from AllowUsers — account never logged in). Inventory the unrestricted server-to-server keys in deploy's authorized_keys (`deploy@professeer` and `deploy@coppermind` on fishery; `deploy@coppermind` and `lsd-to-professeer` on valaritas; `siddarth@reasonablesolutions.in` and `qa-server-deploy` on coppermind): each either gets a forced command pinned to its actual job (rsync path, dbsync) or gets deleted. Today any one compromised box can shell into the others as deploy.
17. **waystone single-user:** `sudo deluser --remove-home ubuntu` (stock image account — password locked, authorized_keys empty, instance metadata holds keys for siddarth only, so the guest agent won't recreate it), then `gcloud compute instances add-metadata vm-dev-siddarth --zone=europe-west3-b --metadata=block-project-ssh-keys=TRUE`. Periodically audit project IAM: whoever holds `compute.instanceAdmin` / metadata write / `iap.tunnelResourceAccessor` can mint users on this box regardless of /etc/passwd.
18. **su/sudo audit retention:** on fishery, valaritas, coppermind, give `/var/log/auth.log` its own logrotate stanza (`monthly`, `rotate 12`) instead of the shared 4-week default in `/etc/logrotate.d/rsyslog`. While there: `sudo systemctl restart rsyslog` so log lines stop stamping the stock image hostname (`Ubuntu-2404-noble-amd64-base`) and carry `fishery`/`valaritas` — an audit trail that can't say which box it's from is half a trail.

---

*The May process didn't fail because it was wrong — it failed because development had nowhere to live except the servers, and nothing ever surfaced drift. This version gives development a home (dev boxes), makes linkage cheap (squash PRs), makes servers incorruptible (read-only keys + deploy gates), and looks in the mirror weekly (hygiene report). The rest is honor — which is fine, as long as the mirror is honest.*
