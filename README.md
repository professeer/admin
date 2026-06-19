# professeer / admin

Org-level resources for the **professeer** GitHub organization: process guides,
reusable document templates, and brand assets.

## Contents

| Folder | What's inside |
|--------|---------------|
| [`guides/`](guides/) | Team playbooks — development process, GitHub project setup, release cadence. |
| [`templates/`](templates/) | Reusable document templates. Currently: the branded LaTeX letterhead. |
| [`logos/`](logos/) | Brand logo kit — lockup + mark, in crimson (light bg) and white (dark bg), SVG + PNG. See [`logos/README.md`](logos/README.md). |

## Guides

- [`guides/20260610_devops_process_professeer_v1.md`](guides/20260610_devops_process_professeer_v1.md) — development process (branch flow, reviews).
- [`guides/github-project-setup-management.md`](guides/github-project-setup-management.md) — GitHub org + project onboarding playbook.
- [`guides/RELEASE.md`](guides/RELEASE.md) — release cadence.

## Templates

- [`templates/theprof-letterhead/`](templates/theprof-letterhead/) — branded one-page LaTeX letterhead (crimson header band, owl footer, Manrope type). Self-contained; see its `README.md` for build steps. Compile with **LuaLaTeX**.
- [`templates/theprof-academic-presentation/`](templates/theprof-academic-presentation/) — branded Beamer slide deck (crimson accents, owl mark, frame-numbered footer). Blank skeleton; see its `README.md`. Compile with **pdfLaTeX**.
