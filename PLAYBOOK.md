# Script and Automation Development Methodology

**Internal document — Data Team**

A practical guide to standardize the day-to-day development of automation scripts and report generation in Multilog's DWH environment.

| Field | Detail |
|---|---|
| Audience | Data and automation team |
| Scope | Python scripts, SQL, Power Automate, n8n, Task Scheduler |
| Environment | Azure SQL (dev_bronze/silver/gold), AWS EC2 Windows server |
| Version | 1.0 — 04/23/2026 |

---

## Philosophy: Data Engineering Lite

Heavyweight methodologies like formal Scrum or SAFe don't fit the workflow of a small data team, where requirements arrive out of cycle and an urgent task can interrupt any two-week plan. But the complete absence of methodology leads to technical chaos: untested scripts, hardcoded credentials, direct changes in production, and no way to audit what was done or when.

The solution is a middle ground: **personal Kanban** to manage work without unnecessary ceremonies, combined with **software engineering practices** applied with common sense to the world of data. The process fits in five phases that repeat in a cycle, with a clear boundary between work in development and work in production.

### The principle that sums it all up

Write boring and predictable code, follow the same pattern in every script, always validate against a source of truth, and never modify code directly in production. Discipline is not the enemy of speed: it's what makes speed sustainable over time.

### Why these five phases

Each phase exists to solve a specific problem that, if skipped, eventually shows up as a production incident:

| Phase | Problem it prevents |
|---|---|
| 1. Plan | Starting to code without knowing exactly what needs to be done. |
| 2. Develop | Scripts with no common structure that no one can maintain. |
| 3. Validate | Code that runs without errors but produces wrong numbers. |
| 4. Review | Changes with no history or explanation that get lost over time. |
| 5. Deploy and operate | Silent failures in production that no one catches in time. |

---

## The development cycle

A task always moves in the same order. Each transition only happens if the previous phase's criteria are met. The weekly retrospective turns lessons learned in prod into process improvements.

```
[DEV]
  Plan  →  Develop  →  Validate  →  Review and merge
                                           ↓
[PROD]                              Deploy and operate
            ↑                                |
            └──────── Weekly retro ──────────┘
```

> **Note on the dev/prod boundary.** The transition between phase 4 and phase 5 is not decorative. It marks the crossing from experimental work (where breaking something is cheap) to production (where an error can affect reports, workflows, or business decisions). The standard of care is different on each side of that boundary.

---

## 01 — Plan

> DEV work

Most scripts that become a problem in production didn't fail at execution — they failed because the requirement was never clearly defined. The planning phase exists so that when development starts, the problem and the acceptance criteria are already settled.

### Kanban board structure

Five columns, no more, no less. The specific tool (GitHub Projects, Trello, Excel) matters less than the board being the single source of truth for all tasks.

| Column | Limit | What it contains |
|---|---|---|
| Backlog | No limit | Unrefined ideas, requirements, and bugs. |
| Next | 5–7 cards | Refined cards, ready to be worked on. |
| In progress | 2 cards | Active work. Low WIP intentionally. |
| In review | 2 cards | Code awaiting validation or PR. |
| Done | No limit | Merged and deployed. Archived weekly. |

### How to prioritize

Each card answers two questions: how much it hurts if you don't do it this week (impact) and how much it will cost to do it (effort on a 1–3 scale: 1 = hours, 2 = a day or two, 3 = several days). If you estimate 3, the card isn't ready yet — it needs to be split. Priority order: first 1s of high impact (quick wins), then 2s and 3s of high impact (the real work), then low-impact 1s.

### Why the WIP limit of 2

Low WIP isn't about doing less work — it's about forcing yourself to finish before starting. With five tasks in parallel, each one progresses at 20% and nothing gets delivered. With two, one finishes this week and delivers value. Also, a low WIP exposes bottlenecks: a card stuck in review for four days becomes visible on the board instead of being hidden behind new work.

### Anatomy of a good card (before moving to Next)

- [ ] Actionable title (`"Add duplicate validation"`, not `"Problem with sales"`)
- [ ] Context in one sentence: what problem it solves and for whom
- [ ] Explicit acceptance criterion: how we know it's done
- [ ] Effort estimate: 1, 2, or 3 (never 3 without splitting first)
- [ ] Target environment: dev first, prod after

---

## 02 — Develop

> DEV work

The goal is not to write brilliant code: it's to write boring and predictable code that follows the same structure as every other script in the repo. In six months, anyone — including your future self — must be able to read it without context.

### Git workflow

Branching model, commit conventions, and commit language are defined in [CONTRIBUTING.md](CONTRIBUTING.md). In short: always branch off `dev`, open PRs toward `dev`, commits in English using conventional commits (`feat:`, `fix:`, `refactor:`).

**Golden rule:** if a Kanban card requires two separate branches, it's two separate cards.

### The template as a contract

For new scripts, copy `scripts/_template` and adapt it. The structure is always the same:

| File | Responsibility |
|---|---|
| `README.md` | Purpose, inputs, outputs, frequency, how to debug |
| `config.py` | Load environment variables and dev/prod validation |
| `db.py` | DWH connections with context manager |
| `queries.py` | Parameterized SQL with `Template` (never concatenate) |
| `transforms.py` | Pure business logic, no I/O (testable) |
| `logger.py` | Logging to console and date-rotated file |
| `main.py` | Orchestration with `--dry-run` and `--check` flags |
| `tests/` | Unit tests for `transforms.py` |

### The development loop

Write a pure function, write its unit test before continuing, run `pytest`, commit. Repeat. Small commits, one per completed logical piece. If the commit message needs the word "and," the commit is too large.

**Practical rule:** connect to the real world (DB, APIs, files) only once the logic in `transforms.py` is working and tested with mock DataFrames. Starting with the connection locks you into 10-second iteration cycles; starting with pure logic lets you iterate in milliseconds.

### Avoid premature polish

The common temptation is to refactor, abstract, and add options before the script actually works. The correct order is: **first make it work, then make it correct, then make it clean.** Most scripts reach "works and is correct" and that is enough.

---

## 03 — Validate

> DEV work

Code can run without errors and still produce wrong results. A script that silently outputs incorrect numbers is more dangerous than one that crashes — at least a crash is visible. Validation is the gate that catches data errors before they reach any downstream consumer.

### Four levels of validation

Run in order, from cheapest to most expensive. If one fails, there's no point moving to the next until it's fixed.

| Level | What it validates | How |
|---|---|---|
| Style and lint | Formatting and conventions | Run all pre-commit hooks (see [CONTRIBUTING.md](CONTRIBUTING.md)). SQLFluff for SQL. |
| Unit tests | Functions in `transforms.py` | `pytest` passes at the root of the full repo. |
| Functional dry-run | End-to-end integration | `--dry-run` against dev. Coherent log and logical counts. |
| Against source of truth | Data correctness | Totals match Tableau/AXIS/SAP with documented tolerance. |

### Validation against source of truth

This is the most important validation and the one most often skipped. The central question: do my script's numbers match the numbers people see today? Before replacing an existing process, you must demonstrate equivalence (or document that the new one fixes a real bug in the old one). Define tolerance (e.g., 1%) and document the comparison in the PR with concrete numbers.

### Checklist before opening the pull request

Complete the [PR Checklist in CONTRIBUTING.md](CONTRIBUTING.md#pr-checklist). Additionally, for data scripts:

- [ ] `pytest` passes at the root of the full repo
- [ ] SQL validated with SQLFluff if applicable
- [ ] `--check` confirms DWH connectivity
- [ ] `--dry-run` runs without errors, coherent log
- [ ] Log counts match manual queries
- [ ] Validation against source of truth documented with numbers
- [ ] The script is idempotent (run twice, same result)

---

## 04 — Review and merge

> DEV work

The pull request matters regardless of team size. Writing and reviewing are incompatible cognitive modes — the PR is the mechanism that forces a deliberate switch between them, and the structure of a good PR is what makes that switch effective.

### Let the code rest

The most useful and hardest rule to adopt: don't open the PR the moment you finish coding. Let a few hours pass — ideally the whole night. With fresh eyes you'll notice confusing names, logic that seemed clear last night but now doesn't, stale comments. If you can't wait until the next day, at least 20–30 minutes of rest before reviewing.

### Anatomy of a good PR

| Section | What it must contain |
|---|---|
| Title | Actionable, with prefix (`feat/fix/refactor`). Example: `"feat: detect duplicates"`. |
| What changes | Behavior before vs. after, in prose. Do not describe the code. |
| Why | Reason for the change. Link to the Kanban card. |
| How to test | Concrete steps you yourself executed when validating. |
| Risks | What could go wrong, what's out of scope, what decisions you made. |

### The five self-review questions

Reviewing file by file in the GitHub interface (not in the local editor, because the editor hides the diff):

1. **Does it do what the PR says it does?** "While I was at it" changes not mentioned in the description are a red flag: either document them or remove them from the PR.
2. **Do the names communicate intent?** Variables like `df`, `data`, `temp`, and functions like `process`, `handle`, `update` are opaque when re-read.
3. **What happens in the edge case?** Empty DataFrame, columns with nulls, future date, all records duplicated.
4. **Are the logs enough to diagnose at 3 AM?** If the error only says "failed" with no context or traceback, it's insufficient.
5. **Is there dead code, TODO items without cards, stale comments?** Either clean it up or turn it into a backlog card.

### Before merging: final verification

Before opening the PR, run the [AI Review Prompt](README.md) on your code and apply the relevant suggestions. Then verify:

```bash
git rebase dev && pytest && python -m scripts.name.main --dry-run
```

If any of those three fail, go back to Develop. No exceptions: `"it's a minor fix, I'll do it after the merge"` is exactly the pattern that makes `dev` stop being deployable.

Squash commits and merge into `dev` following the [PR Checklist in CONTRIBUTING.md](CONTRIBUTING.md#pr-checklist). Delete the feature branch after the merge.

---

## 05 — Deploy and operate

> Production

The dev/prod boundary is crossed. Until this point, an error was contained. From here on, an error can affect reports, billing, or downstream processes that other people depend on. The standard of care is higher and must be applied deliberately.

### Deployment sequence

1. Connect via RDP to the DWH-Multilog server.
2. `git pull origin main` in `C:\Scripts-Multilog`.
3. If there are new dependencies: activate venv and `pip install -r requirements.txt`.
4. Verify that `.env` has `ENVIRONMENT=prod` (the flow 0122 incident was exactly this).
5. Run `python -m scripts.name.main --check` to validate connectivity.
6. Run `python -m scripts.name.main` in full, without dry-run, with real data, monitoring the live log.
7. Schedule in Task Scheduler or n8n. Explicit timezone `America/Mexico_City` (UTC-6).
8. Monitor the first scheduled execution live.
9. Update the README with the exact time, task name, and owner.

### Monitoring: three levels

| Level | When | Purpose |
|---|---|---|
| Morning review | Every day, first 10 min | Look for `WARNING`/`ERROR` in overnight logs. |
| Active alerts | Critical scripts only | Email, WhatsApp, or Healthchecks.io when something fails. |
| Monthly review | First Monday of the month | Trends, slow scripts, log rotation, general health. |

### Protocol when something fails

- **Stabilize, don't fix.** If the report failed, the priority is to deliver output (run manually if needed), not to debug.
- **Preserve the evidence.** Copy the failure log before modifying anything. Save partial files and state.
- **Investigate in dev, not in prod.** Replicate the case locally. Fix, test, open a normal PR. The methodology applies equally under pressure.
- **Restore and monitor.** Once the fix is on main and deployed, observe the next execution live.
- **Short postmortem.** `docs/postmortems/YYYY-MM-DD-name.md` with: what happened, impact, root cause, actions to prevent recurrence.

### Production server rule

> **Never modify code directly on the production server.** Not a one-line change, not a temporary test. The prod server is a mirror of `main`, not an independent source. All changes follow the same path: local → feature branch → validation → PR → merge → `git pull` on the server. Without this, the rest of the process cannot be trusted.

---

## Anti-patterns worth naming

Each of these patterns is the root cause of at least one real production incident. Recognizing them by name makes it easier to avoid them and to correct a teammate (or yourself) when they appear.

| Anti-pattern | Description |
|---|---|
| Code on the server | Modifying directly on the server instead of going through git. Results in divergence between `main` and what's actually running, impossible to audit. |
| Hardcoded credentials | Passwords, tokens, or specific paths inside the code. They must live in `.env`, outside the repo, loaded by `config.py`. |
| Wrong environment | Scripts pointing to prod when they should be in dev or vice versa. Preventable with explicit `ENVIRONMENT` validation at the start of each execution. |
| Unlimited WIP | Five cards "in progress" and none finished. Priorities shift, context is lost, nothing gets delivered. |
| Filler tests | Tests that always pass because they don't test anything real. Worse than having no tests, because they create a false sense of security. |
| Adjusted validation | Changing the `assert` to make the test pass instead of investigating why it failed. The most common way to silence real bugs. |
| Print in production | Using `print()` instead of `logging`. Prints disappear when the script runs in the background, leaving you with no evidence when something fails. |
| PR without description | Pull request titled "changes" with an empty description. Impossible to review, impossible to audit in the future. |
| Fix and move on | `"It's a minor fix, I'll do it after the merge."` `main` stops being deployable and discipline breaks down silently. |
| Logs without counts | Logs that only say "process finished" without numbers. Makes it impossible to detect trends (script getting slower, counts dropping). |

---

## The three non-negotiable disciplines

Everything else in this document is a consequence or application of these three. If something has to give under pressure, it should not be any of these.

### 1. All changes go through git, a branch, and a PR

No direct changes on the server. No direct commits to `dev` or `main`. Branch off `dev`, open a PR, merge with squash. Without this discipline, the code on GitHub and the code that runs diverge, and auditability disappears.

### 2. Strict dev / prod separation

The `ENVIRONMENT` variable controls everything. It is validated explicitly at the start of every script. No script touches prod without having run successfully in dev first.

### 3. Validate against a source of truth before publishing

It's not enough for the code to run. The numbers must match, with documented tolerance, before any consumer uses them to make a decision.

---

## Practical next step

This document is a reference, not a plan. To make it operational:

- Clone the base repository with the `_template` template and the working example.
- Copy the 5-column Kanban board in GitHub Projects (or the preferred tool).
- Migrate existing scripts one by one, starting with the simplest, to bring them up to the standard.
- Institutionalize the Friday retro: 30 minutes, three questions, cards to the backlog.
- Review this document every six months and update it as the team's practices evolve.