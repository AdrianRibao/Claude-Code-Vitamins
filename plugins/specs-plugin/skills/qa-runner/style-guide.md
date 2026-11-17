# QA Style Guide

Conventions for the **QA plan** that bugfix-writer, task-writer and tdd-writer put in their specs, and for the **QA record** that the `qa-runner` skill writes after each run. One guide, so a bug doc, a task file and a UI TDD describe QA the same way.

## Contents

- When a spec has a QA plan
- QA plan structure
- Environments
- Logins
- Checks
- Capabilities (closed list)
- Exploratory charter
- QA record
- Results
- Anti-patterns

## When a Spec Has a QA Plan

| The change                        | QA plan                                                                                                                                                            |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Touches UI                        | **Required.** Anything a user sees or does in the browser: screen, component, form, navigation, copy, layout                                                       |
| API or backend only               | **None.** No browser QA: behavior is covered by unit tests, and an API bug fix is still confirmed at the live endpoint. Verification says `QA: n/a — no UI change` |
| Both                              | The plan covers the UI; API behavior stays in unit tests                                                                                                           |
| Operational (deploy, config, DNS) | None, unless the outcome is checked through a UI                                                                                                                   |

- **TDDs are judged by content, not template type.** Any TDD with UI sections — including the default combined TDD — has a QA Plan; a TDD without UI has none and drops the QA criteria from Testing.
- **PRDs never carry a QA plan**: they have no environments or implementation. The TDD writer turns a PRD's user workflows into the TDD's checks.

## QA Plan Structure

One section with four parts in this order: **Environments**, **Logins**, **Checks**, **Exploratory charter**. Bug docs and task files write `## QA plan` and `### Exploratory charter`; TDDs title-case them (`## QA Plan`, `### Exploratory Charter`). The [template](templates/qa-plan.md) has the skeleton.

The plan says WHAT to check and what passing looks like — never an agent-browser command. Commands change between releases; the runner maps capabilities to commands at run time and keeps them in `commands.log`.

## Environments

| Env     | Base URL                      | Start / deploy                    | Seed data                          |
| ------- | ----------------------------- | --------------------------------- | ---------------------------------- |
| local   | `http://localhost:4000`       | `bin/dev`                         | `bin/seed-qa` (3 records, 1 admin) |
| staging | `https://staging.example.com` | PR -> CI -> `staging_deploy_hold` | `qa-*` accounts, nightly reset     |

- **local** is always present; it is the pre-commit run.
- **staging** only for projects that have one — omit the row otherwise. It runs after deploy, before promotion to production.
- Never production.
- A Base URL cell may list several origins, comma-separated, when the change spans apps (a widget and a POS); every login must land on one of them.

## Logins

| Role     | local            | staging            |
| -------- | ---------------- | ------------------ |
| admin    | `qa-admin-local` | `qa-admin-staging` |
| staff    | `qa-staff-local` | `qa-staff-staging` |
| customer | `anonymous`      | `anonymous`        |

- **One profile per role and environment.** An agent-browser vault profile stores a single login URL, so a profile saved for local cannot log in to staging. Drop the staging column when the project has no staging.
- `anonymous` means no login.
- Optional: a `Logged in when` line under the table names, per role, a text or element visible only after login. Without it, leaving the login URL is the signal.
- **Never a credential in the spec** — no passwords, tokens or cookies, and no commands. The user saves each profile once in the agent-browser vault, in their own terminal; the spec only names it.
- Test accounts only. A profile that points at a real person's account is a defect in the plan.

## Checks

```markdown
| #    | Covers | Check                                        | Capability          | Passes when                                          | Evidence            |
| ---- | ------ | -------------------------------------------- | ------------------- | ---------------------------------------------------- | ------------------- |
| QA-1 | AC-1   | Admin archives a record and confirms         | `dialog`, `network` | Row shows "Archived"; `PATCH /records/:id` -> 200 once | screenshot, request log |
| QA-2 | AC-1   | Admin archives a record and cancels          | `dialog`, `network` | Row unchanged; no `PATCH` sent                       | request log         |
| QA-3 | —      | Records list accessibility                   | `a11y`              | No serious or critical violations at WCAG 2.1 AA     | audit report        |
```

- **`#`**: `QA-1`, `QA-2`, … permanent within the spec.
- **Covers**: the acceptance criterion it proves (`AC-N`, or a TDD criterion), or `—` for a cross-cutting check (accessibility, performance).
- **Check**: one user-observable behavior, imperative, with the role doing it.
- **Capability**: one or more from the [closed list](#capabilities-closed-list).
- **Passes when**: observable and specific — a visible text, a URL, a request and its status, a count, a threshold. Never "works", "looks right", or "no errors" without saying where.
- **Evidence**: one or more of screenshot, page snapshot, video, request log, console log, audit report, vitals report, diff image.
- **Scope cross-cutting checks to the screens the change touches.** They judge the screen as it is, not only the diff: a violation that predates the change fails the check and is fixed in the change, so an app-wide audit would turn one change into a cleanup project.
- A bug doc's plan always includes the check that replays the **original reproduction steps** and confirms the symptom is gone, plus one check per blast-radius item that is visible in the UI.

## Capabilities (Closed List)

| Capability      | Checks that…                               | agent-browser feature                                     | `Passes when` example                                                 |
| --------------- | ------------------------------------------ | --------------------------------------------------------- | --------------------------------------------------------------------- |
| `ui-flow`       | A journey renders and responds             | snapshot + click/fill, wait for text or URL, `is` / `get` | Toast "Saved" visible; URL is `/records/:id`                          |
| `dialog`        | Each branch of a confirm or prompt         | `dialog` status / accept / dismiss                        | Cancel leaves the row unchanged                                       |
| `network`       | The UI sends or receives the right request | `network requests` filtered by URL, method, status        | `PATCH /records/:id` -> 200, exactly once                             |
| `console-clean` | No uncaught errors during the flow         | `errors`, `console`                                       | Zero page errors and zero console errors during QA-1                  |
| `a11y`          | Automated accessibility audit              | `a11y` (axe-core) with WCAG tags, scoped to a selector    | No serious or critical violations at WCAG 2.1 AA on `/checkout`       |
| `responsive`    | Layout at a viewport or device             | `set viewport` / `set device`                             | Records render as cards at 375×812, no horizontal scroll              |
| `visual`        | Layout matches the approved baseline       | `diff screenshot --baseline`                              | Under 1 % of pixels differ from `specs/qa/baselines/records-list.png` |
| `vitals`        | Performance budget                         | `vitals`                                                  | LCP < 2.5 s and CLS < 0.1 on staging                                  |

The runner takes the syntax from `agent-browser skills get core`, `skills get core --ref commands` and `<command> --help` — this table names features, not commands. Rules:

- **`dialog`**: one row per branch — a confirm has an accept row and a dismiss row.
- **`network`** observes. Mocking or blocking requests is allowed only on local, and only when the row says so; on staging that row is `skipped — mocks requests`.
- **`visual`**: the budget in `Passes when` is a share of differing pixels. agent-browser's `--threshold` is a per-pixel colour tolerance, not that budget. Baselines live in `specs/qa/baselines/`, are committed, and are approved by the user the first time the runner creates them.
- **`vitals`** gates only on staging. On local the result is `informational`; in a project without staging, write `informational — no staging` as the row's `Passes when`, since it can never gate.
- A new capability is a change to this list (and to `qa-runner`), not something a spec invents.

## Exploratory Charter

One or two lines that scope agent-browser's dogfood pass: the changed screens, their neighbours, the roles to explore as, and the risks worth probing.

> Records list and record detail, as admin and staff. Probe: empty list, 1 vs 100+ records, long names, archiving the last record on a page, mobile width.

Every UI change has a charter. The pass runs on local only.

## QA Record

Written by the runner under the plan — `### QA record` in bug docs and task files, `### QA Record` in TDDs — with one block per environment:

- A full run replaces that environment's block.
- An `--only` re-run updates only the named rows and keeps the other rows and the Exploratory line.

```markdown
### QA record

**local · 2026-11-17 · `http://localhost:4000` · `task/012-archive-records` at `HEAD 3f2a9c1`, code `9b1e04d2` · agent-browser 0.43.0**

| #    | Result | Observed                                              | Evidence              |
| ---- | ------ | ----------------------------------------------------- | --------------------- |
| QA-1 | pass   | "Archived" shown; one `PATCH /records/42` -> 200      | `qa-1.png`, `qa-1-requests.json` |
| QA-2 | pass (after fix) | First run sent the `PATCH` on cancel; fixed in `archive_button.ex:31` with a regression test, re-run clean | `qa-2-requests.json` |

**Exploratory**: 3 findings — fixed: pagination reset to page 1 after archiving the last row of page 2 (pre-existing, on the records list this task changes; `records_live.ex:88`, regression test added, re-run clean); filed: bug 019 (record detail's "Back" link drops the list's sort order — pre-existing, on a screen this task does not touch); filed: task 013 (undo after archive — an improvement, not a defect).

**Evidence**: `specs/qa/runs/012-archive-records/2026-11-17-local/` (git-ignored)
```

- **Observed** says what was seen, not "as expected". A fixed row names the fix (`file:line`) and says when the defect predates the change. A filed finding says why it is outside the change: the screen it is on.
- The build line names what was tested: on local, the branch, `HEAD` and the code fingerprint the runner computes, so `--resolve` and `--ship` can tell whether code changed since; on staging, the deployed version.
- A waiver appears only when the user stated one, quoted with its date.

## Results

| Result                                | Meaning                                                                                              | Gate accepts                                                           |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `pass`                                | Observed what `Passes when` says                                                                     | Yes                                                                    |
| `pass (after fix)`                    | Failed, the defect was fixed in this change (even a pre-existing one), re-run clean                  | Yes                                                                    |
| `pass (baseline approved YYYY-MM-DD)` | First `visual` run; the user approved the new baseline                                               | Yes                                                                    |
| `informational`                       | `vitals` on local; the numbers are recorded, not judged                                              | Yes, for `vitals` on local only                                        |
| `deferred by user — bug NNN`          | The user declined a fix QA asked about; the defect is filed and their words are quoted with the date | Yes, only with the user's quoted words                                 |
| `skipped — <reason>`                  | The rules forbid the capability in this environment (a row that mocks requests, on staging)          | Yes, for that reason only — never because agent-browser is unavailable |
| `baseline created — needs approval`   | Waiting for the user to approve a new baseline                                                       | No                                                                     |
| `fail`                                | Not observed                                                                                         | No                                                                     |

## Anti-Patterns

| Anti-pattern                                                          | Why it fails                                                                    |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| agent-browser commands in the spec                                    | Break on the next release; the spec is WHAT, the runner is HOW                  |
| "Works" or "looks right" as a pass condition                          | Cannot fail, so it proves nothing                                               |
| A screenshot as the only proof                                        | Shows a moment, not that the request was sent or the error absent               |
| Credentials, tokens or cookies in the spec                            | Leak through git and transcripts; name a vault profile instead                  |
| One login profile for local and staging                               | The profile's saved URL logs in to one environment only                         |
| An app-wide cross-cutting check                                       | Every old violation fails it and must be fixed; scope it to the changed screens |
| Filing a defect on a screen the change touches                        | The changed screen ships with a known defect; fix it in the change              |
| Fixing a pre-existing defect on an untouched screen inside the change | Mixes unrelated changes into one commit; file it with `/bugfix`                 |
| Loosening a `Passes when` the app fails                               | Turns a failing check into a pass without fixing anything                       |
| QA as a real user, or on production                                   | Real data, real side effects; test accounts on local and staging only           |
| Staging run without checking the build                                | A stale deploy passes every check                                               |
| A QA plan for an API-only change                                      | Duplicates unit tests with a slower, flakier harness                            |
