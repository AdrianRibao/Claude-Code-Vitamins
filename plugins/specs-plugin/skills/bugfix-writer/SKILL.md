---
name: bugfix-writer
description: Generate bugfix specifications — symptom, reproduction, root cause, scoped fix plan, and a red->green regression test — and optionally drive the fix end to end. Use when documenting a bug, planning a fix, reproducing a defect, or verifying a fix works without regressions.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - TodoWrite
  - Task
  - Bash
  - WebFetch
  - WebSearch
  - mcp__plugin_context7_context7__resolve-library-id
  - mcp__plugin_context7_context7__query-docs
---

# Bugfix Writer Skill

> **Philosophy**: A bugfix is a *delta* — the gap between observed and expected behavior. The spec's job is to close that gap with the smallest correct change, proven by a test that fails on the bug and passes on the fix. Diagnose the root cause, never the symptom.

## When to Use

- A reported defect needs a written, reproducible specification before anyone fixes it
- Planning a fix: capturing symptom, reproduction, root cause, and a scoped change
- Reproducing a defect and proving it with a failing (red) regression test
- Driving a fix end to end (`--fix`): red test -> fix -> green -> no regressions
- Reviewing an existing bug doc for missing root cause, reproduction, or regression test
- Consolidating a bug doc after open decisions are answered

## Usage

```
/bugfix [name] [--ticket ID] [--page URL] [--from @path] [--fix] [--branch NAME] [--no-branch] [--ask] [--review @path] [--consolidate @path] [--resolve @path]
```

| Flag                  | Purpose                                                                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `--ticket ID`         | Ingest the bug report from a tracker issue (Jira/Linear/GitHub) via MCP                                                                 |
| `--page URL`          | Ingest the bug report from a spec page (Confluence/Notion) via MCP or WebFetch                                                          |
| `--from @path`        | Ingest the bug report from a local file or pasted text                                                                                  |
| `--fix`               | Execute the fix loop: prove the regression test red, apply the fix, prove green, run the suite                                          |
| `--branch NAME`       | Use/create this branch for the fix; default auto-creates `fix/<ticket-or-NNN>-<slug>` when on a protected branch                        |
| `--no-branch`         | Apply the fix on the current branch; do not create or switch branches                                                                   |
| `--ask`               | Force surfacing of non-blocking decisions for review (default: proceed silently)                                                        |
| `--review @path`      | Analyze an existing bug doc for gaps (no root cause, no repro, no red->green test)                                                      |
| `--consolidate @path` | Apply answered decisions, collapse to a Decisions table, tighten the doc                                                                |
| `--resolve @path`     | Mark a confirmed bug fixed: set `Status` to `fixed` and `git mv` the doc into `specs/bugs/fixed/` (refuses unless the fix is confirmed) |

## Output Location

Bug docs are created at:

```
specs/bugs/{NNN}-{slug}.md
```

On a confirmed fix the file moves to:

```
specs/bugs/fixed/{NNN}-{slug}.md
```

**Conventions:**

- `{NNN}` is a zero-padded 3-digit number, sequential and **per-repo** (each repo starts at `001`). Scan `specs/bugs/` and `specs/bugs/fixed/` for the highest existing number and add one.
- `{slug}` is a kebab-case descriptor of the defect.
- H1 title format: `# Bug NNN — <description with backticked identifiers> (<parenthetical scope>)`.
- Open bugs live at the top of `specs/bugs/`; resolved bugs move into `specs/bugs/fixed/`.

Ask the user to confirm or override the path during Phase 0.

## Resources

- [style-guide.md](style-guide.md) - Bugfix writing conventions: metadata block, severity scale, root-cause discipline, do-NOT-change guardrails, AC format
- [templates/bugfix.md](templates/bugfix.md) - The bugfix document template
- [examples/](examples/) - Reference bug docs (UI fix and backend fix)

## Behavioral Mindset

Evidence first: reproduce the bug and capture concrete evidence (real IDs, URLs, payloads, dates) before theorizing. Trace the symptom to the actual defect in code with `file:line` provenance — a root cause is a causal explanation, never a restatement of the symptom. Keep the fix as small as the bug demands and name explicitly what must NOT change. Every bugfix is anchored by one regression test that fails on the buggy code and passes on the fix.

**Proceed by default; do not stall on questions.** Unlike a PRD or TDD for a brand-new feature, most bugs arrive well-specified. Only stop to ask when a *correct* fix is genuinely impossible without the answer. For everything else, choose the sensible option, record it as a documented assumption, and keep moving. See [Question Policy](#question-policy).

## Key Actions

For a `--fix` run (and as a planning aid otherwise), copy this checklist and track progress. Steps tagged `(--fix)` run only when fixing.

```
Bugfix Progress:
- [ ] 0. Ingested the report + extracted symptom and expected behavior
- [ ] 0. Reproduced with concrete evidence (or recorded a hypothesis)
- [ ] 0. Assigned severity; asked only blocking questions
- [ ] 1. Traced symptom -> actual defect (file:line + provenance)
- [ ] 2. Specified the fix; listed Do-NOT-change + Out-of-scope; mapped blast radius
- [ ] 2. Designed the regression test (states why it fails on the buggy code)
- [ ] 3. (--fix) On a dedicated fix branch (auto-created if on a protected branch)
- [ ] 3. (--fix) Proved the regression test RED on unfixed code (output recorded)
- [ ] 3. (--fix) Applied the smallest fix; proved it GREEN
- [ ] 3. (--fix) Ran unchanged-behavior tests + suite; no regressions
- [ ] 4. Created unit + E2E tests at the levels the fix warrants; all green
- [ ] 4. (--fix) Confirmed the ORIGINAL issue is gone in the running app (browser for UI)
- [ ] 4. Wrote verification + manual demo mapped to acceptance criteria
- [ ] 5. Triaged for blockers; emitted `## Open decisions` only if blocked
- [ ] 6. (--fix) Staged the verified fix + doc move; did NOT commit — handed off to the user
- [ ] 6. Presented for review; on confirmed fix, staged the move to fixed/ + Status update for the user's fix commit
```

### Phase 0: Intake & Reproduce (Always)

1. **Ingest the report**: Pull the bug from its source:
    - `--ticket ID` -> use whatever tracker MCP tools are connected (Jira/Linear/GitHub). A ticket is often a thin pointer — follow smartlinks to the real spec page.
    - `--page URL` -> use the connected wiki MCP (Confluence/Notion) or WebFetch.
    - `--from @path` -> read a local file or pasted text.
    - No source -> work from the user's prompt.
2. **Extract the essentials**: symptom, expected behavior, affected area/repo, reporter, date, environment, and any attached evidence.
3. **Reproduce deterministically**: Run the steps, hit the URL/endpoint, seed the data. Capture the *actual* broken behavior as evidence (output, payload, screenshot reference).
4. **Triage severity**: Assign a severity (see [style-guide.md](style-guide.md)) and note user/business impact.
5. **Ask only blocking questions** (see [Question Policy](#question-policy)): if the expected behavior is genuinely undefined or a product/policy fork changes the fix, use `AskUserQuestion`. Otherwise proceed.

**Reproduction gate**: do not specify a fix until the bug is reproduced, OR the doc explicitly records "not reproducible — proceeding by hypothesis" with the hypothesis stated.

### Phase 1: Root Cause Analysis

1. **Trace symptom -> defect**: Use Grep/Glob/Read (and Task for broad search) to follow the behavior to the line(s) of code responsible.
2. **Identify third-party surfaces**: If the defect involves a library/SDK/API/CLI/service, fetch current docs (context7 -> WebFetch -> WebSearch) before asserting how it behaves. Never diagnose a third-party behavior from memory.
3. **Write the causal chain**: state the actual defect with `file:line` and provenance (how/where verified, with date). Distinguish symptom from cause.

**Root-cause gate**: the documented root cause must be a causal explanation ("the label builder falls back to a positional index when dates collide"), not a paraphrase of the symptom ("labels show (1)(2)(3)").

### Phase 2: Fix Plan, Blast Radius & Regression Test Design

1. **Specify the fix**: WHAT changes — file/function, the behavior delta, signatures or ≤5-line snippets only (never full implementations).
2. **Bound the scope**: list **Do NOT change** items and **Out of scope** explicitly. Bugs invite "while I'm here" creep — reject it.
3. **Map blast radius**: enumerate adjacent behavior the change could break and what must remain unchanged.
4. **Design the regression test**: the one test that *captures the bug* — describe what it asserts and why it fails on the current (buggy) code. This is the load-bearing artifact.
5. **Design unchanged-behavior tests**: assertions that the "Do NOT change" items still hold after the fix.

### Phase 3: Fix Execution (only with `--fix`)

1. **Select the working branch** (before changing any code):
    - `--branch NAME` -> create/switch to it.
    - On a protected branch (`main`/`master`/`develop`/`release/*`) -> create `fix/<ticket>-<slug>` (fall back to `fix/<NNN>-<slug>` when no ticket) and switch; announce the branch.
    - Already on a feature branch -> use it as-is.
    - `--no-branch` -> stay on the current branch.
    - Record the chosen branch in the doc's `Related` and `Deploy & rollback`.
2. **Prove red**: write the regression test and run it against the unfixed code. Confirm it FAILS for the expected reason. Record the failure output.
3. **Apply the fix**: make the smallest change that addresses the root cause.
4. **Prove green**: re-run the regression test. Confirm it PASSES.
5. **Prove no regressions**: run the unchanged-behavior tests and the surrounding suite. If anything breaks, fix and re-run from step 3 — repeat until the regression test is green AND the suite passes.
6. **Record evidence**: capture red output, green output, and suite result in the doc.

Without `--fix`, this phase is written as a plan (red expectation + fix + verification), executed later by the implementer.

### Phase 4: Verification & Demo

1. **Automated tests at the right levels** (create them if missing — do not stop at the single regression test):
    - **Unit tests** for the changed logic, branch, or validation.
    - **E2E tests** for the user-facing flow where the bug was visible (UI journey, API contract) when the fix warrants it.
    - **Unchanged-behavior tests** for each blast-radius item.
    - Run them all; every test must pass.
2. **Confirm the issue in the running app**: reproduce the ORIGINAL steps and confirm the symptom is actually gone — passing tests alone are not proof. For UI bugs, drive a real browser (e.g. the `claude-in-chrome` browser skill); for APIs, hit the live endpoint. Capture evidence (screenshot / response).
3. **Manual / How to demo**: step-by-step reproduction-then-confirmation, with ✅ expected and ❌ failure signals, mapped to acceptance criteria (P11 demo style).
4. **Trace**: every acceptance criterion maps to a symptom and to a test.

### Phase 5: Blocking-Question Triage (replaces heavyweight Open Questions)

1. **Invoke Sequential MCP** with `--ultrathink` to scan for genuine fix-blockers and unconsidered edge cases.
2. **Classify** each finding (see [Question Policy](#question-policy)): blocking, default-and-document, or non-issue.
3. **Emit sparingly**: only blocking items become an `## Open decisions` section. Default-and-document items become inline **Assumptions**. If nothing blocks, write no open-decisions section.

### Phase 6: Finalize & Lifecycle

1. **Verify before any commit**: a fix counts as "confirmed" only when all [Seven Non-Negotiable Gates](#the-seven-non-negotiable-gates) hold — tests green AND the original issue confirmed gone in the running app. **Do not commit (or advise committing) while any gate is unmet.**
2. **Present for review**: show the complete bug doc; await approval before any implementation (when not `--fix`).
3. **Stage the verified change, then hand off**: once all seven gates pass, on the `fix/...` branch apply the fix, `git mv` the doc from `specs/bugs/` into `specs/bugs/fixed/`, set its `Status` to `fixed` (fill in the `— PR #NNN, commit <hash>` reference when you commit), and `git add` everything so the fix + doc move are **staged together**. **The skill does not commit** — you review the staged diff and make the commit / open the PR. The move rides in that fix commit and reaches `main` only when the PR merges. Preserve the `NNN`.
4. **Quality check**: run the [Quality Checklist](#quality-checklist) and `mdformat` on the doc.

## The Seven Non-Negotiable Gates

These are the bugfix analog of the TDD permission-matrix gate. A bug doc is **not acceptable**, and a `--fix` change **must not be committed**, until all seven hold (or are explicitly, justifiably waived in the doc):

| Gate                             | Requirement                                                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Reproduction**                 | The bug is reproduced with concrete evidence, OR the doc records "not reproducible — hypothesis: …"                            |
| **Root cause, not symptom**      | The root cause names the actual defect with `file:line` and provenance — not a restatement of the symptom                      |
| **Red->green regression test**   | Exactly the test that fails on the buggy code and passes on the fix; with `--fix`, the RED state is demonstrated, not assumed  |
| **Tests at the right levels**    | Unit tests cover the changed logic and E2E tests cover the user-facing flow, created where the fix warrants them — all green   |
| **Blast radius / Do-NOT-change** | Adjacent behavior that could break is enumerated and covered by unchanged-behavior tests                                       |
| **Confirmed fixed in the app**   | The original issue is reproduced and confirmed gone in the running app (drive a browser for UI bugs) — not inferred from tests |
| **Scope discipline**             | Explicit In/Out of scope; the fix is the smallest change that addresses the root cause                                         |

> A green regression test that was never shown to fail on the buggy code proves nothing — it may pass for reasons unrelated to the fix. And passing tests are necessary but not sufficient: a bug is "fixed" only once the original issue is reproduced and confirmed gone in the running app. **The skill never commits while any gate is unmet** — with `--fix` it stops at a verified, staged change and hands off.

## Question Policy

**Default: proceed.** Asking a teammate to review and answer a question introduces real latency, and most bugs don't need it. Classify every potential question into one of three tiers:

| Tier                     | When                                                                                                                               | Action                                                                                               |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Blocking**             | A *correct* fix is impossible without the answer (expected behavior undefined; a product/policy fork changes which code you touch) | Stop and ask via `AskUserQuestion` (Phase 0) or record under `## Open decisions`                     |
| **Default-and-document** | Minor uncertainty (an edge case, a naming choice) where a sensible default exists                                                  | Choose the default, record it as an inline **Assumption** ("Assumed X — flag if wrong"), keep moving |
| **Non-issue**            | Fully specified                                                                                                                    | Say nothing; do not manufacture questions                                                            |

**Irreversible-fix guardrail**: when the fix touches data loss, data migration, money, or security, the blocking bar drops — confirm the approach before proceeding even if the spec looks complete.

`--ask` overrides the default and surfaces default-and-document items as explicit decisions, for the rare gnarly bug where you want a second opinion before fixing.

## Include in Bug Docs

| Element             | Purpose                                                                | Example                                                                                             |
| ------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Metadata block      | Status, Severity, Reported, Area, Repo, Related (bold-key bullets)     | `**Status:** open` … `**Severity:** Major — customer-facing`                                        |
| Summary             | What's broken, who's hit, + a one-line scope statement                 | "UI-only fix to the dropdown label string; no data changes"                                         |
| Symptom             | Concrete, reproducible observed behavior                               | Real IDs/URLs, expected-vs-actual, before/after blocks                                              |
| Reproduction        | Deterministic steps, seed/restore scripts, environment                 | Shell seed script + step-by-step navigation                                                         |
| Root cause          | The actual defect with `file:line` + provenance                        | "`labels.js:42` falls back to positional index on date collision (verified on staging, 2026-06-23)" |
| Expected behavior   | The correct behavior, as numbered rules when needed                    | "Rule 1: shared date+time -> append (HH:mm Option X)"                                               |
| The fix             | WHAT changes + **Do NOT change** + **Out of scope**                    | "Replace index fallback with group-based label; do NOT touch dedup logic"                           |
| Blast radius        | Adjacent behavior at risk; what must stay unchanged                    | "Stock/places-left labels must render identically"                                                  |
| Verification        | Regression test (red->green) + unchanged-behavior tests + manual demo  | "Test fails on `(1)(2)(3)`, passes on `Option A/B/C`"                                               |
| Acceptance criteria | Table (`# / Context / Action / Expected`) + mirrored `- [ ]` checklist | "AC-1 … / `- [ ] AC-1: …`"                                                                          |
| Open decisions      | Only genuine blockers (else omit)                                      | "OD-01: should Option letters be stable across edits?"                                              |
| Deploy & rollback   | When deploy is non-trivial                                             | "CircleCI -> staging_deploy_hold; rollback = revert PR"                                             |
| When fixed (footer) | Update Status, move to `fixed/`                                        | "Set Status to fixed and `git mv` into `specs/bugs/fixed/`"                                         |

## Exclude from Bug Docs

| Element                  | Why Exclude                                     | Where It Belongs                         |
| ------------------------ | ----------------------------------------------- | ---------------------------------------- |
| Full fix implementation  | The diff lives in the PR, not the spec          | The code / PR                            |
| Symptom-as-root-cause    | Hides the real defect; produces a band-aid      | (must be replaced with the actual cause) |
| Unrelated cleanups       | Scope creep; dilutes the regression test        | A separate change                        |
| Manufactured questions   | Stalls a ready fix for no reason                | (omit)                                   |
| New-feature requirements | A bug restores intended behavior, not new scope | A PRD/TDD                                |

## Acceptance Criteria Format (hybrid)

Bug docs use **both** a readable table and an `ac-checker`-parseable checklist, under one `## Acceptance Criteria` heading:

1. **Table** for humans/QA: columns `# | Context | Action | Expected result`, IDs `AC-1`, `AC-2`, … Each row traces to a symptom.
2. **Mirrored checklist** so `ac-checker` can parse and auto-mark: one `- [ ] AC-N: <one-line>` per table row, plus the regression-test criterion.

The regression-test criterion is mandatory: `- [ ] Regression test fails on the buggy code and passes on the fix`.

## Lifecycle Management

The skill owns the bug's lifecycle so the `fixed/` convention is applied consistently:

- **Branch** (`--fix` only): work lands on a dedicated `fix/<ticket-or-NNN>-<slug>` branch — auto-created when the current branch is protected (`main`/`master`/`develop`/`release/*`), reused when already on a feature branch, overridable with `--branch` / `--no-branch`.
- **Status values**: `open` -> `in progress` -> `fixed — PR #NNN, commit <hash> (<one-line>)`.
- **The move is staged with the fix, not committed by the skill**: once all seven gates pass on the `fix/...` branch, `git mv` the doc into `specs/bugs/fixed/`, set `Status` to `fixed`, and `git add` it — leaving fix + doc move **staged together** (preserve the `NNN`). **You** make the commit / open the PR; the move rides in that fix commit and reaches `main` only after it merges.
- **Evidence bar**: never mark a bug `fixed` or move it until all [Seven Non-Negotiable Gates](#the-seven-non-negotiable-gates) hold — a passing regression test alone is not enough.
- **`--resolve @path`** (explicit close command): verifies the evidence bar, sets `Status` to `fixed`, and `git mv`s the doc into `specs/bugs/fixed/` — **staged, not committed** (you make the commit). Use it when the fix was made outside a `--fix` run (`--fix` already stages this in Phase 6). Refuses if the fix is not confirmed.

## MCP Integration

- **Sequential MCP**: blocking-question triage (Phase 5) and structured root-cause reasoning.
- **Context7 MCP**: current docs for any third-party surface involved in the defect.
- **Tracker / wiki MCP** (Jira, Confluence, Linear, Notion, GitHub — whatever is connected): ingest the bug report for `--ticket` / `--page`. These are user-connected servers; if none is available, ask the user to paste the report or use `--from`.

## Outputs

- **Bug doc** (default): symptom, reproduction, root cause, scoped fix plan, regression-test design, verification plan, acceptance criteria.
- **Fixed bug** (`--fix`): all of the above plus demonstrated red->green evidence, unit + E2E tests, the issue confirmed fixed in the running app, a passing suite, updated `Status`, and the file moved to `fixed/`.
- **Bug doc review** (`--review`): gap analysis against the seven gates (missing repro, symptom-as-cause, no red->green test, missing unit/E2E coverage, no app confirmation, unbounded scope).
- **Consolidated bug doc** (`--consolidate`): answered decisions applied, collapsed into a Decisions table, tightened.
- **Resolved bug** (`--resolve`): `Status` set to `fixed` and the doc moved into `specs/bugs/fixed/` (evidence bar enforced).

## Examples

### Document a bug from a ticket

```
/bugfix variant-dropdown-labels --ticket OP-6979

# Pulls the Jira ticket, follows the smartlink to the Confluence spec,
# reproduces, diagnoses root cause, writes specs/bugs/NNN-variant-dropdown-labels.md
```

### Reproduce, fix, and verify end to end

```
/bugfix online-product-count --from @notes/bug.md --fix

# Reproduces, writes a red regression test, applies the fix, proves green,
# runs the suite, sets Status=fixed, moves the file to specs/bugs/fixed/
```

### Review an existing bug doc for gaps

```
/bugfix --review @specs/bugs/002-seasonal-pricing.md

# Flags: root cause restates the symptom; no red->green regression test; scope unbounded
```

## Quality Checklist

Before finalizing a bug doc, verify:

- [ ] Metadata block present: Status, Severity, Reported, Area, Repo, Related
- [ ] **Reproduction is deterministic with concrete evidence (or an explicit "not reproducible — hypothesis" note)**
- [ ] **Root cause names the actual defect with `file:line` and provenance — not a restatement of the symptom**
- [ ] Expected behavior is stated (as numbered rules when non-trivial)
- [ ] The fix specifies WHAT changes, with no code block longer than 5 lines
- [ ] **`Do NOT change` and `Out of scope` are explicit**
- [ ] **Blast radius enumerated; adjacent behavior covered by unchanged-behavior tests**
- [ ] **A regression test is specified that fails on the buggy code and passes on the fix**
- [ ] **With `--fix`: the RED state was demonstrated (failure output recorded), then GREEN, then the suite passed**
- [ ] **Unit tests and E2E tests created at the levels the fix warrants — all green**
- [ ] **With `--fix`: the original issue was reproduced and confirmed GONE in the running app (browser for UI bugs) — not inferred from tests**
- [ ] **No commit was made (or advised) until all seven gates hold**
- [ ] Acceptance criteria present as BOTH a table and a mirrored `- [ ]` checklist (ac-checker compatible)
- [ ] **The regression-test acceptance criterion is present in the checklist**
- [ ] Any third-party behavior was verified against freshly fetched docs, not memory
- [ ] No manufactured questions; only genuine fix-blockers appear under `## Open decisions`
- [ ] Default-and-document choices recorded as inline Assumptions
- [ ] On a confirmed fix: the doc is `git mv`d into `specs/bugs/fixed/` and `Status` set to `fixed`, **staged (not committed)** so it rides in the fix commit the user makes
- [ ] `mdformat` passes on the document

## Boundaries

**Will:**

- Ingest a bug report from a tracker, a wiki page, a file, or the prompt
- Reproduce the defect and capture evidence before specifying a fix
- Diagnose the root cause with `file:line` provenance
- Specify the smallest correct fix and bound its scope explicitly
- Design (and with `--fix`, execute and prove) a red->green regression test
- Proceed by default, asking only genuine fix-blocking questions
- In `--fix`, work on a dedicated `fix/...` branch — auto-created when on a protected branch
- Add unit and E2E tests at the levels the fix warrants
- Confirm the original issue is gone in the running app (browser for UI) before declaring it fixed
- Manage the bug lifecycle (Status + move to `fixed/`)

**Will Not:**

- Fix symptoms while leaving the root cause in place
- Include full implementation code in the spec (the diff belongs in the PR)
- Expand scope into unrelated cleanups or new features
- Block a well-specified, ready-to-fix bug on non-essential questions
- Mark a bug fixed without evidence (passing regression test, merged PR, or explicit confirmation)
- Make any git commit — with `--fix` it branches, applies + verifies the fix, and **stages** the change (fix + doc move); you review and commit
- **Advise committing a fix before all seven gates hold: unit + E2E tests green AND the issue confirmed resolved in the running app**
- Edit code unless `--fix` is passed
