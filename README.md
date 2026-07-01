# Claude Code Vitamins

A plugin marketplace for [Claude Code](https://claude.ai/code) containing reusable skills and commands to enhance your development workflow.

## Available Plugins

### specs-plugin

Generate excellent Product Requirements Documents (PRDs), Technical Design Documents (TDDs), and bugfix specifications with structured workflows and deep analysis. Verify implementation completeness against TDD acceptance criteria.

| Command                    | Skill         | Purpose                                                                                           |
| -------------------------- | ------------- | ------------------------------------------------------------------------------------------------- |
| `/specs-plugin:prd`        | prd-writer    | Generate PRDs focused on problems, goals, users, and success metrics                              |
| `/specs-plugin:tdd`        | tdd-writer    | Generate TDDs focused on requirements, contracts, and acceptance criteria                         |
| `/specs-plugin:bugfix`     | bugfix-writer | Diagnose and optionally fix bugs — symptom, root cause, red->green regression test, verify in app |
| `/specs-plugin:ac-checker` | ac-checker    | Verify acceptance criteria are implemented with tests and code                                    |

### marketing-board

Eight-seat marketing deliberation board for SaaS launches. Convenes 8 specialist subagents in parallel against a launch brief and synthesizes a Marketing Plan Memo you iterate on with `--consolidate`.

| Command / Skill              | Purpose                                                                                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------------- |
| `/marketing-board`           | Convene the board on a brief. 8 specialists deliberate in parallel; output is a Marketing Plan Memo. |
| `/marketing-board:bootstrap` | Scaffold the product knowledge base (`agent_docs/marketing/`) the board reads before deliberating.   |

**Specialists**: Ethnographer, Storyteller, Channel Strategist, Producer, Funnel Engineer, Community/PR Lead, Moonshot, Contrarian. Each runs in its own context window and returns a structured contribution; the synthesizer reconciles them into a coherent plan.

**Install**:

```bash
/plugin install marketing-board@claude-code-vitamins
```

**Usage**:

```bash
# 1. Bootstrap product context (one-time per product)
/marketing-board:bootstrap

# 2. Run the board on a launch brief
/marketing-board "Launching <product> in <market>. Goal: <metric>."

# 3. Iterate on the memo
/marketing-board --consolidate @marketing-plans/<slug>-<date>.md
```

## Requirements

- Claude Code 2.1.0 or later
- **Optional MCP servers** (for enhanced functionality):
    - [Context7](https://github.com/upstash/context7-mcp) - Library documentation lookups
    - Sequential Thinking - Deep analysis for Open Questions generation

## Installation

### 1. Add the Marketplace

In Claude Code, add this marketplace:

```bash
# From GitHub (recommended)
/plugin marketplace add AdrianRibao/Claude-Code-Vitamins

# Or from any git URL
/plugin marketplace add https://github.com/AdrianRibao/Claude-Code-Vitamins.git
```

### 2. Install a Plugin

```bash
# Install specs-plugin
/plugin install specs-plugin@claude-code-vitamins
```

### 3. Verify Installation

```bash
# Test the commands
/specs-plugin:prd --help
/specs-plugin:tdd --help
```

### Keeping Plugins Updated

**Manual update:**

```bash
# Refresh marketplace and update plugins
/plugin marketplace update
```

**Enable auto-updates (recommended):**

Auto-update is disabled by default for third-party marketplaces. To enable automatic updates:

1. Run `/plugin` to open the plugin manager
2. Select the **Marketplaces** tab
3. Choose **claude-code-vitamins** from the list
4. Select **Enable auto-update**

When enabled, Claude Code will automatically refresh the marketplace and update installed plugins at startup.

> **Note**: To disable all automatic updates globally, set the environment variable `DISABLE_AUTOUPDATER=true`

## Usage

### PRD Writer

Generate Product Requirements Documents focused on problems, goals, users, and success metrics.

```bash
# Create a master PRD
/specs-plugin:prd time-tracking --type master

# Create a feature PRD
/specs-plugin:prd time-tracking-mobile --type feature

# Review existing PRD for scope creep
/specs-plugin:prd --review @specs/prds/time-tracking/00-master.md

# Consolidate after answering Open Questions
/specs-plugin:prd --consolidate @specs/prds/time-tracking/01-mobile.md
```

**Flags:**

| Flag                  | Purpose                                               |
| --------------------- | ----------------------------------------------------- |
| `--type`              | PRD type: `master`, `feature`, `api`, `integration`   |
| `--no-questions`      | Skip upfront questions (use with comprehensive brief) |
| `--review`            | Analyze existing PRD for scope creep                  |
| `--consolidate @path` | Apply OQ answers, tighten document                    |

**Output:** `specs/prds/{product}/{nn}-{product}-{type}.md`

### TDD Writer

Generate Technical Design Documents focused on requirements, contracts, and acceptance criteria.

```bash
# Create a backend TDD
/specs-plugin:tdd incidents --type backend --prd @specs/prds/02-incident-management.md

# Create a UI TDD
/specs-plugin:tdd incidents-ui --type ui --prd @specs/prds/02-incident-management.md

# Review existing TDD for bloat
/specs-plugin:tdd --review @specs/tdds/incidents/01-incident-resource.md

# Consolidate after answering Open Questions
/specs-plugin:tdd --consolidate @specs/tdds/incidents/01-incident-resource.md
```

**Flags:**

| Flag                  | Purpose                                             |
| --------------------- | --------------------------------------------------- |
| `--type`              | TDD type: `backend`, `ui`, `api`, `integration`     |
| `--prd @path`         | Reference PRD for requirements                      |
| `--no-questions`      | Skip upfront questions (use with comprehensive PRD) |
| `--review`            | Analyze existing TDD for bloat                      |
| `--consolidate @path` | Apply OQ answers, tighten document                  |

**Output:** `specs/tdds/{feature}/{nn}-{feature}-{type}.md`

### Bugfix Writer

Document a bug (symptom, reproduction, root cause, scoped fix, red->green regression test) and optionally drive the fix end to end.

```bash
# Document a bug from a Jira ticket (follows smartlinks to the spec page)
/specs-plugin:bugfix variant-dropdown-labels --ticket OP-6979

# Reproduce, fix, and verify end to end (auto-branches, runs tests, confirms in app)
/specs-plugin:bugfix online-product-count --from @notes/bug.md --fix

# Review an existing bug doc for gaps (no root cause, no red->green test)
/specs-plugin:bugfix --review @specs/bugs/002-seasonal-pricing.md

# Close a confirmed bug: set Status to fixed and move it to specs/bugs/fixed/
/specs-plugin:bugfix --resolve @specs/bugs/003-online-product-count.md
```

**Flags:**

| Flag                                          | Purpose                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------- |
| `--ticket ID` / `--page URL` / `--from @path` | Ingest the bug report from a tracker, wiki page, or local file            |
| `--fix`                                       | Reproduce -> red test -> fix -> green -> unit/E2E tests -> confirm in app |
| `--branch NAME` / `--no-branch`               | Override branch selection (default: auto-branch on protected branches)    |
| `--ask`                                       | Surface non-blocking decisions for review                                 |
| `--review @path`                              | Analyze an existing bug doc for gaps                                      |
| `--consolidate @path`                         | Apply answered decisions, tighten the doc                                 |
| `--resolve @path`                             | Mark a confirmed bug fixed: update Status and move to `specs/bugs/fixed/` |

**Output:** `specs/bugs/{nnn}-{slug}.md` -> `specs/bugs/fixed/{nnn}-{slug}.md` once resolved

### Acceptance Criteria Checker

Verify that acceptance criteria from TDDs are actually implemented in code with tests and proper coverage.

```bash
# Check implementation status
/specs-plugin:ac-checker specs/tdds/incidents/01-incidents-backend.md

# Check with coverage analysis
/specs-plugin:ac-checker specs/tdds/user-auth/01-user-auth-backend.md --coverage

# Check and auto-update TDD checkboxes
/specs-plugin:ac-checker specs/tdds/dashboard/02-dashboard-ui.md --update

# Compare against specific branch
/specs-plugin:ac-checker specs/tdds/incidents/01-incidents-backend.md --branch develop
```

**Flags:**

| Flag         | Purpose                                                |
| ------------ | ------------------------------------------------------ |
| `--coverage` | Run coverage analysis to verify targets                |
| `--update`   | Update TDD by marking implemented criteria as complete |
| `--branch`   | Compare against specific branch (default: main)        |

**What it checks:**

- ✅ Tests exist for each criterion (searches test files)
- ✅ Implementation code exists (searches source files)
- ✅ Coverage targets met (≥80% domain, ≥90% business rules, ≥95% auth)
- ✅ Checkbox status matches implementation

**Output:** `specs/tdds/reports/ac-implementation-{feature}-{timestamp}.md`

## Workflow

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  1. CREATE PRD                                                               │
│     /specs-plugin:prd feature --type master                                  │
│                                                                              │
│     • Define problem, goals, users, requirements                             │
│     • Generate Open Questions via deep analysis                              │
│     • Review and consolidate                                                 │
└──────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌──────────────────────────────────────────────────────────────────────────────┐
│  2. CREATE TDD                                                               │
│     /specs-plugin:tdd feature --type backend --prd @specs/prds/feature.md    │
│                                                                              │
│     • Define data models, contracts, acceptance criteria                     │
│     • Generate Open Questions via deep analysis                              │
│     • Review and consolidate                                                 │
└──────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌──────────────────────────────────────────────────────────────────────────────┐
│  3. IMPLEMENT                                                                │
│                                                                              │
│     • PRD and TDD are complete and approved                                  │
│     • Write tests and implementation code                                    │
│     • Iteratively check progress with ac-checker                             │
└──────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌──────────────────────────────────────────────────────────────────────────────┐
│  4. VERIFY                                                                   │
│     /specs-plugin:ac-checker specs/tdds/feature.md --coverage --update       │
│                                                                              │
│     • Validate tests exist for all acceptance criteria                       │
│     • Verify implementation code is complete                                 │
│     • Check coverage targets are met                                         │
│     • Update TDD checkboxes, generate report                                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Bugfix Workflow

The bugfix skill runs a defect from report to closed. With `--fix` it will not let an unproven fix be committed — every gate must hold first.

```
┌───────────────────────────────────────────────────────────────────────────┐
│  1. INTAKE & REPRODUCE                                                      │
│     /specs-plugin:bugfix labels --ticket OP-6979                           │
│     • Pull the report (ticket / wiki / file / prompt); reproduce w/ evidence│
│     • Triage severity; ask only genuinely fix-blocking questions           │
│     -> writes specs/bugs/{NNN}-{slug}.md            Status: open            │
└───────────────────────────────────────────────────────────────────────────┘
                                     ↓
┌───────────────────────────────────────────────────────────────────────────┐
│  2. DIAGNOSE                                                                │
│     • Trace symptom -> the actual defect at file:line (cause, not symptom) │
│     • Scope the fix (Do-NOT-change + Out-of-scope); design red->green test  │
└───────────────────────────────────────────────────────────────────────────┘
                                     ↓
┌───────────────────────────────────────────────────────────────────────────┐
│  3. FIX  (--fix)                                                           │
│     /specs-plugin:bugfix labels --ticket OP-6979 --fix                     │
│     • Auto-branch fix/<ticket-or-NNN>-<slug> if on a protected branch       │
│     • Prove test RED -> apply fix -> prove GREEN; add unit + E2E tests      │
│                                                     Status: in progress    │
└───────────────────────────────────────────────────────────────────────────┘
                                     ↓
┌───────────────────────────────────────────────────────────────────────────┐
│  4. VERIFY — the Seven Gates   (COMMITS FORBIDDEN until all seven hold)    │
│     reproduction · root cause · red->green · unit+E2E · blast radius ·      │
│     confirmed-in-app · scope discipline                                    │
│     • Confirm the ORIGINAL issue is gone in the running app                │
│       (drive a browser for UI bugs, e.g. claude-in-chrome)                 │
└───────────────────────────────────────────────────────────────────────────┘
                                     ↓
┌───────────────────────────────────────────────────────────────────────────┐
│  5. LAND & CLOSE   (in the fix PR)                                         │
│     • Commit the fix on the branch; in the SAME PR: git mv the doc into     │
│       specs/bugs/fixed/ and set Status: fixed — PR #NNN, commit <hash>      │
│     • Lands on main when the PR merges                                      │
└───────────────────────────────────────────────────────────────────────────┘
```

### Updating status and moving a bug to `fixed/`

A bug doc starts at `specs/bugs/{nnn}-{slug}.md` with `Status: open`. It moves to `specs/bugs/fixed/` **only once the fix is confirmed** — all seven gates hold (including a real-app confirmation, not just green tests). Two ways to make that transition:

**Automatic** — a `--fix` run does it for you in the final step: after the gates pass it sets the `Status` line to `fixed` and `git mv`s the doc into `specs/bugs/fixed/`, all inside the fix PR.

**Explicit close** — when the fix was made outside a `--fix` run, resolve the doc in one command:

```bash
/specs-plugin:bugfix --resolve @specs/bugs/003-online-product-count.md
```

It verifies the fix is confirmed, then updates the metadata block and moves the file:

```
- **Status:** open           ->  - **Status:** fixed — PR #517, commit 9c1a4f0 (...)
specs/bugs/003-...-count.md   ->  specs/bugs/fixed/003-...-count.md   (git mv, NNN kept)
```

**Manual equivalent** (same effect, if you prefer to do it by hand):

```bash
# 1. Edit the metadata block: Status: fixed — PR #NNN, commit <hash> (one-line)
# 2. Move the file (preserve the number), committed inside the fix PR:
git mv specs/bugs/003-online-product-count.md specs/bugs/fixed/003-online-product-count.md
```

> Never move a doc to `fixed/` until the seven gates hold — a passing test alone is not proof the bug is fixed.

## Philosophy

### PRD: WHAT and WHY

PRDs define **what problem we're solving** and **why it matters**. They focus on:

- Problem statement and impact
- Goals and non-goals
- Target users and their needs
- User workflows (step-by-step scenarios)
- Requirements (functional and non-functional)
- Success metrics

**PRDs never include**: implementation code, database schemas, API specifications, architecture decisions.

### TDD: WHAT to Build

TDDs specify **what needs to be built** in technical terms. They focus on:

- Data models (tables, not code)
- Interface contracts (signatures only)
- Authorization matrices
- Behavior specifications (Given/When/Then)
- Acceptance criteria (testable checkboxes)

**TDDs never include**: full implementation code, function bodies, algorithm details, test implementations.

### Bugfix: Diagnose and Prove

Bugfix specs close the gap between observed and expected behavior with the smallest correct change, proven by a test that fails on the bug and passes on the fix. They focus on:

- Symptom and reproduction (concrete evidence)
- Root cause at `file:line` (never the symptom)
- A scoped fix with explicit Do-NOT-change / Out-of-scope
- A red->green regression test plus unit and E2E coverage
- Confirmation the issue is gone in the running app

**Bugfix specs never ship an unproven fix**: commits are forbidden until the seven gates pass — including a real-app confirmation, not just green tests.

## Document Structure

### PRD Structure

| Section           | Purpose                                  |
| ----------------- | ---------------------------------------- |
| Problem Statement | Why this product/feature exists          |
| Goals & Non-Goals | What success looks like, what's excluded |
| Target Users      | Who we're building for, their needs      |
| User Workflows    | Step-by-step scenarios for user tasks    |
| Requirements      | Functional and non-functional needs      |
| Success Metrics   | How we measure success                   |
| Scope             | What's in/out of scope                   |
| Open Questions    | Unresolved decisions with IDs            |

### TDD Structure

| Section              | Purpose                                 |
| -------------------- | --------------------------------------- |
| Overview & Scope     | What's included/excluded                |
| Data Model           | Attributes, types, constraints (tables) |
| Interface Contract   | Function signatures only (no bodies)    |
| Authorization Matrix | Who can do what (tables)                |
| Behavior Specs       | Given/When/Then scenarios               |
| Acceptance Criteria  | Testable checkboxes                     |
| Open Questions       | Unresolved decisions with IDs           |

### Bugfix Structure

| Section             | Purpose                                         |
| ------------------- | ----------------------------------------------- |
| Metadata            | Status, Severity, Reported, Area, Repo, Related |
| Summary             | What's broken, who's hit, scope statement       |
| Symptom             | Observed vs expected, concrete evidence         |
| Reproduction        | Deterministic steps / seed                      |
| Root cause          | The defect at `file:line` + provenance          |
| Expected behavior   | The correct behavior (rules)                    |
| The fix             | WHAT changes + Do-NOT-change + Out of scope     |
| Blast radius        | Adjacent behavior at risk                       |
| Verification        | Regression + unit/E2E + manual/app demo         |
| Acceptance criteria | Table + mirrored checkbox list                  |

## Open Questions

Both PRDs and TDDs generate Open Questions via deep analysis (Sequential MCP + ultrathink):

```markdown
## Open Questions

| ID    | Question                              | Status         |
| ----- | ------------------------------------- | -------------- |
| OQ-01 | Should we support multiple employers? | Open           |
| OQ-02 | Maximum file upload size?             | Deferred to v2 |

### OQ-01: Multiple Employers

**Question**: Should an employee work for multiple employers?

**Why it matters**: Affects data model, UI complexity, reporting.

**Possible answers**:

- Single employer (simpler, v1)
- Multiple employers (broader market)

**Status**: Open - needs product decision
```

## Splitting Large Documents

### PRD Thresholds

| Indicator       | Threshold         | Action                |
| --------------- | ----------------- | --------------------- |
| User workflows  | > 8 workflows     | Split by user segment |
| Requirements    | > 30 requirements | Split by feature area |
| Document length | > 1000 lines      | **Must split**        |

### TDD Thresholds

| Indicator           | Threshold       | Action           |
| ------------------- | --------------- | ---------------- |
| Data models         | > 3 resources   | Split by domain  |
| Acceptance criteria | > 25 checkboxes | Split by feature |
| Document length     | > 1500 lines    | **Must split**   |

## Contributing

### Adding a New Skill

1. Create skill directory: `plugins/{plugin}/skills/{skill-name}/`
2. Add `SKILL.md` with frontmatter and definition
3. Add `style-guide.md` with writing conventions
4. Add `templates/` with document templates
5. Add `examples/` with reference examples
6. Create command wrapper in `commands/{command}.md`

### Adding a New Plugin

1. Create plugin directory: `plugins/{plugin-name}/`
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills and commands
4. Register in `.claude-plugin/marketplace.json`

## License

MIT
