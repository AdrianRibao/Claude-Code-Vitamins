---
name: tdd-writer
description: Generate lean Technical Design Documents focused on requirements, contracts, and acceptance criteria. Use when creating TDDs, converting PRDs to specs, or documenting backend resources, APIs, or UI components.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - TodoWrite
  - Task
  - WebFetch
  - WebSearch
  - mcp__plugin_context7_context7__resolve-library-id
  - mcp__plugin_context7_context7__query-docs
---

# TDD Writer Skill

> **Philosophy**: TDDs specify WHAT to build, not HOW. Implementation details belong in code, not documentation.

## When to Use

- New feature or domain requiring technical specification
- Backend resource, API, or UI design documentation needs
- Requirements clarification before implementation
- Converting PRD requirements into actionable technical specs
- TDD review requests to identify and remove code bloat
- Consolidating TDD after Open Questions are answered

## Usage

```
/tdd [feature-name] [--prd @path] [--no-questions] [--review] [--consolidate @path] [--type backend|ui|api|integration]
```

| Flag                  | Purpose                                             |
| --------------------- | --------------------------------------------------- |
| `--prd @path`         | Reference PRD for requirements                      |
| `--no-questions`      | Skip upfront questions (use with comprehensive PRD) |
| `--review`            | Analyze existing TDD for bloat                      |
| `--consolidate @path` | Apply OQ answers, tighten document                  |
| `--type`              | Optional. Split TDD by type when complexity demands |

## Output Location

By default, a single combined TDD is created:

```
specs/tdds/{feature}/{nn}-{feature}.md
```

**Examples:**

| Command                        | Output Path                                   |
| ------------------------------ | --------------------------------------------- |
| `/tdd user-auth`               | `specs/tdds/user-auth/01-user-auth.md`        |
| `/tdd user-auth --prd @prd.md` | `specs/tdds/user-auth/01-user-auth.md`        |
| `/tdd user-auth --type api`    | `specs/tdds/user-auth/01-user-auth-api.md`    |
| Master TDD (complex features)  | `specs/tdds/user-auth/00-user-auth-master.md` |

**Numbering convention:**

- `00-` Master TDD (overview, document hierarchy — only for split TDDs)
- `01+` Feature TDDs (ordered by priority)

**When to use `--type`**: Only when a feature is complex enough to warrant splitting (see Complexity Management). Most features should use a single combined TDD.

Ask user to confirm or override path during Phase 0 questions.

## Resources

- [style-guide.md](style-guide.md) - TDD writing conventions and rules
- [templates/combined-tdd.md](templates/combined-tdd.md) - Default combined template (backend + UI)
- [templates/](templates/) - Type-specific templates (backend, ui, api, integration) for split TDDs
- [examples/](examples/) - Reference TDD examples

## Behavioral Mindset

Requirements-first approach: every section answers "what" not "how". Use tables for scannable data models and authorization matrices. Write signatures without implementations. Create testable acceptance criteria as checkboxes. Never include code blocks longer than 3-5 lines.

**Always ask clarifying questions before generating a TDD.** Understanding requirements upfront prevents rework and produces better specifications.

## Key Actions

### Phase 0: Requirements Discovery (Always)

1. **Read Context**: Review PRD, existing code, related TDDs
2. **Identify Gaps**: Find ambiguities, missing decisions, unclear scope
3. **Ask Questions**: Use `AskUserQuestion` to clarify before writing
    - Scope boundaries (what's in/out)
    - Key business rules and constraints
    - Data ownership and relationships
    - Authorization model
    - Integration points
4. **Confirm Understanding**: Summarize answers before proceeding

**Skip questioning only if:** User provides comprehensive PRD with `--prd` AND explicitly says "no questions needed".

### Phase 1: TDD Core Generation

1. **Discover**: Read PRD, understand requirements, identify scope boundaries
2. **Map Document Hierarchy**: Identify parent PRD, master TDD, sibling TDDs, and related specs
3. **Analyze Existing Patterns**: Find related TDDs and resources in the codebase for consistency
4. **Identify Third-Party Surfaces**: Enumerate every external library, framework, SDK, API, CLI, or cloud service the feature touches. Fetch current docs (context7 → WebFetch → WebSearch as a locator) **before** writing any section that depends on them. See [Third-Party Integration Docs](#third-party-integration-docs-non-negotiable). Never write a third-party surface from memory.
5. **Structure**: Organize into logical sections following lean TDD template
6. **Specify**: Write requirements, data models, contracts, acceptance criteria in tables — cite fetched docs for every third-party value (method names, endpoints, payload fields, headers, scopes, error codes, quotas)
7. **Link Documents**: Add "Related Documents" section with all relevant PRDs and TDDs
8. **Write TDD**: Save to file with placeholder Open Questions section and a populated `External Dependencies` table

### Phase 2: Open Questions Deep Analysis

1. **Invoke Sequential MCP**: Use `--ultrathink` for maximum depth analysis
2. **Analyze Gaps**: Identify ambiguities, missing decisions, edge cases, scope boundaries
3. **Generate Questions**: Create structured OQ entries with IDs, rationale, and possible answers
4. **Append to TDD**: Update the Open Questions section with generated content

### Phase 3: Finalization

1. **Update References**: Update master TDDs and related documents to reference the new TDD
2. **Validate**: Review against PRD, check completeness, verify testability
3. **Quality Check**: Ensure no code blocks > 5 lines, all criteria testable, all links valid
4. **Present for Review**: Show complete TDD to user, await approval before implementation

## Include in TDDs

| Element                     | Purpose                                                                                                                         | Example                                              |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| Requirements                | WHAT needs to be built                                                                                                          | "Users can create incidents"                         |
| Data Models                 | Attributes, types, constraints (tables)                                                                                         | `hours_adjustment: decimal, required`                |
| Interface Contracts         | Function signatures, action names                                                                                               | `create_incident(params, actor:)`                    |
| Behavior Specs              | Input → Output expectations                                                                                                     | "Creating incident sends notification"               |
| Acceptance Criteria         | How to verify completion                                                                                                        | "[ ] Employee can create incident"                   |
| Testing Requirements        | Test types, coverage targets                                                                                                    | "[ ] Policy tests verify all auth rules"             |
| Constraints & Rules         | Business logic boundaries                                                                                                       | "DL-01: Hours adjustment model"                      |
| Authorization               | Three-layer permission model (tables)                                                                                           | Action, Data, and Condition permissions              |
| Permission Matrix Test Plan | Enumerated PT-xx rows: every cell, every Hidden/R field-role pair, every PC-xx (met + unmet), cross-scope denies, auth boundary | "PT-07: Layer 1, Owner × approve, Deny, expects 403" |
| Related Documents           | Links to PRDs, master TDDs, siblings                                                                                            | "Parent PRD: [link], Backend: [link]"                |
| External Dependencies       | Every third-party library/SDK/API/CLI used, with version, doc URL, fetch date, section consulted                                | "stripe-node v17.4.0, /payment-intents, 2026-05-26"  |
| Open Questions              | Unresolved decisions needing input                                                                                              | "OQ-01: Retention policy?" - Open                    |

### Authorization Model (Three Layers)

TDDs specify authorization through three complementary layers. Not every TDD needs all three — use what fits the feature's complexity. If a feature has no authentication (public pages, anonymous endpoints), skip the authorization section entirely.

**Layer 1 - Action Permissions**: Who can perform which actions, and on what scope. Expand beyond basic CRUD to include domain-specific actions (approve, publish, escalate, export, delegate). Use scope qualifiers: Own, Team, Department, Org, All.

**Layer 2 - Data Permissions**: Which fields each role can see or modify. Use visibility levels: **RW** (read-write), **R** (read-only), **Hidden** (not visible). Only needed when different roles see different fields or have field-level edit restrictions.

**Layer 3 - Permission Conditions**: When permissions apply or are revoked. Captures status-based, time-based, relationship-based, and approval-based constraints. Use rule IDs (PC-01, PC-02) for traceability to acceptance criteria.

See [style-guide.md](style-guide.md) for full format reference and examples.

### Permission Matrix Test Coverage Requirements (non-negotiable)

**The permission matrix is the highest-risk surface in the system. A green test suite that does not exercise every deny path is worse than no tests — it grants false confidence to a reviewer who is about to merge a privilege-escalation bug.** Every TDD whose Authorization section is non-empty MUST include a **Permission Matrix Test Plan** that enumerates one row per required test, and the Acceptance Criteria MUST require that count to be met. Deny coverage is the load-bearing requirement — allow paths are easy to write by accident while shipping a feature; deny paths are not.

**Minimum required test count (the formula)**:

Total required policy tests `N = A + S + H + R + 2·P + E`, where:

- `A` = number of cells in the Layer 1 actor × action matrix (one test per cell — `✅` cells assert the action succeeds within scope, `❌` cells assert explicit Forbidden, never silent failure or no-op).
- `S` = number of scope-qualified `✅ Own / Team / Dept` cells in Layer 1 (one paired cross-scope deny test each: actor attempts the same action on a record **outside** their scope; expected Forbidden).
- `H` = number of `(role, field)` pairs in Layer 2 marked `Hidden` (one test each asserting the field is absent from the response payload — and absent from the DOM for UI surfaces — for that role).
- `R` = number of `(role, field)` pairs in Layer 2 marked `R` read-only (one test each asserting a write attempt is rejected with 422 / changeset error / Forbidden — never silently dropped).
- `P` = number of Layer 3 `PC-xx` permission condition rules (two tests each: one with the condition **met** so the action is allowed, one with the condition **not met** so the action is denied with explicit Forbidden).
- `E` = number of protected entry points (one test each asserting unauthenticated request returns 401 — not 403, not 404; the distinction is part of the contract).

**What counts as a deny test (strict)**:

| Counts                                                                                   | Does NOT count                                                                                                     |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Asserts explicit `403 Forbidden` / `{:error, :forbidden}` / `Pundit::NotAuthorizedError` | Asserts only that result list is empty (could be empty for many reasons unrelated to authorization)                |
| Asserts the unauthorized action did **not** mutate state (row unchanged, no audit event) | Asserts only the HTTP status without checking the side effect didn't happen                                        |
| Asserts the response payload omits the Hidden field                                      | Asserts only that the response was 200 (a 200 with the field still leaked is the exact bug we're guarding against) |
| `404 Not Found` only when explicitly labeled `anti-discovery` in the test plan           | `404` used as a generic deny — it hides the real authorization decision and breaks audit/forensic analysis         |
| Test fixture uses a real authenticated actor in the **denied** role                      | Test that disables auth, stubs the policy, or runs as a superuser to "simplify setup"                              |

**Permission Matrix Test Plan (required artifact)**: every TDD must include this section, generated mechanically from the three layer tables. See [style-guide.md](style-guide.md#permission-matrix-test-plan) for the exact format.

**Acceptance gate**: a TDD is NOT acceptable for implementation until:

1. The Permission Matrix Test Plan section exists and has `≥ formula` rows.
2. Every row has a unique `PT-xx` ID, an explicit `Allow` / `Deny` / `Cross-scope deny` / `Anti-discovery deny` type label, and an explicit expected response.
3. Every `❌` cell in Layer 1 is present as a `Deny` row in the plan.
4. Every `PC-xx` rule appears twice in the plan (one met, one unmet).
5. Every `Hidden` and `R` field-role pair appears in the plan.
6. The Acceptance Criteria > Testing subsection contains the measurable count assertion (e.g., "Policy test count ≥ N, where N = Layer 1 cells (`A`) + 2 × non-RW field-role pairs (`F`) + 2 × PC rules (`R`)").

### UI Test Coverage Requirements (non-negotiable)

**Any TDD that specifies UI changes MUST require end-to-end tests that cover each interactive case.** For every interactive surface — page, form, or live component — tests must exercise the real render/event pipeline of whatever UI framework the project uses and cover every case below. This applies to TDDs of `--type ui` and any combined TDD whose feature includes a UI surface, regardless of language or framework.

| Case             | Required coverage                                                                                                                                                 |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Render           | Page mounts and renders expected markup for each authorized role                                                                                                  |
| Change events    | Every live validation / on-change path, including validation-error branches                                                                                       |
| Submit — success | Every submit success path: redirect, flash/toast, and state change all asserted                                                                                   |
| Submit — failure | Every failure branch: missing field, invalid format, validation rejection, anti-discovery, rate limit, server error                                               |
| Backend effect   | Submitted values actually reach the backend (assert persisted row attributes, job enqueued, message sent, API called with expected payload — not just a redirect) |
| Bug reproduction | Fixing a UI bug requires a test that **fails on the buggy code and passes on the fix**                                                                            |

**Framework examples** (non-exhaustive — use the project's actual stack):

| Stack            | Test tool                                   | Triggers to cover                                              |
| ---------------- | ------------------------------------------- | -------------------------------------------------------------- |
| Phoenix LiveView | `Phoenix.LiveViewTest`                      | `phx-change` / `render_change`, `phx-submit` / `render_submit` |
| Django           | Django test client, Playwright, or Selenium | `GET`/`POST` of form view, HTMX swaps, client submit           |
| React / Next.js  | Testing Library + user-event, Playwright    | `onChange`, `onSubmit`, route transitions                      |
| Rails            | System tests (Capybara)                     | Form fill, submit, validation errors                           |
| FastAPI + HTMX   | `TestClient` or Playwright                  | `hx-post`, `hx-get`, validation swaps                          |

**Tests that only assert a static string survived the request (e.g. page title, header text) after a failed submit do NOT count as coverage** — they pass even when the entire validation pipeline is broken. Acceptance criteria must force assertions on rendered error messages, field-level errors, OR backend state, not on page chrome.

### Third-Party Integration Docs (non-negotiable)

**Any TDD that specifies behavior depending on a third-party library, framework, SDK, API, CLI tool, or cloud service MUST be written against freshly fetched documentation — never from training memory.** Training data ages, vendors break compatibility between minor versions, and a TDD that hardcodes a stale auth flow, endpoint shape, rate limit, or SDK method name will burn the implementer downstream.

**Required before writing any section that references a third-party surface**:

1. **Identify the dependency precisely**: library name + intended version (or "latest stable"), exact API/SDK/CLI surface used. List them before fetching.
2. **Fetch current docs**, in this order of preference:
    - **context7** (`resolve-library-id` then `query-docs`) — primary source for libraries, frameworks, SDKs. Use even for well-known names (React, Stripe, Twilio, AWS SDK, etc.) — your memory may be wrong.
    - **WebFetch** — for vendor service docs, release notes, REST API references, deprecation notices, pricing/quota pages, and anything context7 does not index.
    - **WebSearch** — only to locate the canonical doc URL when you don't already have it; then WebFetch the canonical source.
3. **Cite the source** in the TDD: every third-party dependency must appear in an `External Dependencies` table (see template) with the library/service name, version pinned, doc URL, and the date docs were fetched. A reviewer must be able to re-fetch the same source.
4. **Specify the version**: hardcoded values that depend on SDK or API versions (method names, payload shapes, header names, error codes, quotas, scopes) MUST name the version they were verified against.

**What this rule blocks**:

| Anti-pattern                                                                              | Why it fails                                                                                                       |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| TDD names `stripe.charges.create(...)` from memory                                        | Stripe deprecated `Charges` in favour of `PaymentIntents` years ago — TDD would direct implementer to the dead API |
| TDD specifies an OAuth flow with scopes copied from memory                                | Scopes get renamed and split between versions; a stale scope name causes silent 403s in prod                       |
| TDD assumes a webhook payload shape without re-fetching the vendor's current event schema | Vendors add/rename fields under the same event name; the TDD's "expected payload" is fiction                       |
| TDD pins a quota (rate limit, max payload, retention window) from memory                  | Quotas tighten without warning; the TDD silently encodes a number that is no longer the contract                   |
| TDD describes a CLI tool's flags from memory                                              | Flags get renamed/removed; the runbook the implementer writes against the TDD will not work                        |
| TDD says "see [vendor] docs" with no URL or fetch date                                    | Reviewer cannot verify the spec; implementer must redo the research from scratch                                   |

**Acceptance gate**: a TDD that references any third-party surface is NOT acceptable for implementation until:

1. The `External Dependencies` table exists and lists every third-party library/service/CLI used by the feature.
2. Each row has: name, version (or version range), doc URL, fetch date, and the specific section/method/endpoint consulted.
3. Every method name, endpoint path, payload field, header, scope, error code, or quota stated in the TDD is traceable to an `External Dependencies` row (i.e. the implementer can verify it from the cited source).
4. The Acceptance Criteria > Testing subsection requires contract tests against the **current** vendor contract (e.g. recorded fixtures dated within the freshness window, or a live contract test in CI) — not against assumed shapes.

## Exclude from TDDs

| Element              | Why Exclude                                              | Where It Belongs |
| -------------------- | -------------------------------------------------------- | ---------------- |
| Full module code     | Gets stale, duplicates codebase                          | Implementation   |
| Internal algorithms  | Implementation detail                                    | Code comments    |
| Function bodies      | Will change during dev                                   | Codebase         |
| Boilerplate          | Noise, no specification value                            | Templates        |
| Test implementations | Duplicates test files (but testing requirements belong!) | Test suite       |

## MCP Integration

- **Sequential MCP**: Structured analysis of requirements and systematic TDD construction
- **Context7 MCP**: Framework patterns for data models, authorization patterns
- **Serena MCP**: Project memory for cross-TDD consistency and domain understanding

## Open Questions Generation Workflow

TDD creation follows a **two-step process** where Open Questions are generated separately using deep analysis:

### Step 1: Create TDD Core

Generate all TDD sections EXCEPT Open Questions. For combined TDDs (default), include both backend and UI sections:

1. Overview and Scope
2. Data Model (tables)
3. Interface Contract (signatures only)
4. Authorization (Action Permissions, Data Permissions, Permission Conditions)
5. **Permission Matrix Test Plan** — mechanically enumerate one `PT-xx` row for every cell, cross-scope deny, Hidden/R field-role pair, PC-xx (met + unmet), and auth boundary. The row count MUST satisfy the `N = A + S + H + R + 2·P + E` formula
6. UI: Routes, Screens, Components (if feature has UI)
7. Behavior Specifications (Given/When/Then)
8. Acceptance Criteria (checkboxes with authorization testing — including the measurable test-count assertion referencing the Permission Matrix Test Plan)
9. Related Documents

Write the TDD to file with an empty Open Questions section:

```markdown
## Open Questions

*Generating via deep analysis...*
```

### Step 2: Deep Analysis for Open Questions (ultrathink)

After the TDD core is written, invoke Sequential MCP with `--ultrathink` depth to analyze the complete TDD and generate meaningful Open Questions.

**Analysis Focus Areas**:

| Area                     | Question Types                                                           |
| ------------------------ | ------------------------------------------------------------------------ |
| Requirements Ambiguity   | Unclear acceptance criteria, missing edge cases                          |
| Stakeholder Input Needed | Business decisions, policy clarifications                                |
| Technical Decisions      | Architecture choices, integration approaches                             |
| Scope Boundaries         | What's explicitly out of scope, future considerations                    |
| Data Constraints         | Validation rules, retention policies, limits                             |
| Authorization Gaps       | Missing action permissions, unclear data visibility, untested conditions |

**Self-contained question rule (non-negotiable)**:

Every Open Question must be answerable **without re-reading the TDD**. The reviewer should be able to open the OQ section cold and decide. Concretely:

- **No bare acronyms.** Spell them out on first use in the question, even if defined earlier in the TDD (e.g. write "Permission Condition PC-03 (only owners can edit after approval)" not "PC-03").
- **Inline 1-line context for every reference.** Any mention of a PRD ID, Constraint Rule (CR-xx), Permission Condition (PC-xx), Data Rule (DL-xx), Acceptance Criterion, attribute, or sibling TDD must carry a parenthetical summary of what that thing is.
- **Concrete trade-offs.** Possible answers must describe the real consequence of each choice (cost, latency, UX impact, compliance risk) — not just the value. "90 days — meets SOC2 retention but adds ~40GB/month storage" beats "90 days (standard practice)".
- **State the default the TDD currently implies**, so the reviewer knows what happens if they do nothing.

**Sequential MCP Prompt Template**:

```
Analyze this TDD for unresolved decisions and ambiguities that require
stakeholder input before implementation can begin.

Each question MUST be self-contained — a reviewer should answer it
without re-reading the TDD. Therefore:
- Spell out every acronym on first use (PC-xx, CR-xx, DL-xx, PRD IDs).
- Add a 1-line parenthetical summary whenever referencing another
  rule, attribute, PRD section, or sibling TDD.
- Describe trade-offs concretely (cost, latency, UX, compliance),
  not with labels like "standard practice".
- State the implicit default the TDD currently assumes.

For each question:
1. Assign a unique ID (OQ-01, OQ-02, etc.)
2. State the question as a self-contained sentence
3. Give the 1-line context the reviewer needs to decide
4. Explain why this needs resolution
5. List possible answers with concrete trade-offs
6. Note the current implicit default
7. Mark status as "Open" or "Deferred to v2"

Focus on questions that would BLOCK implementation if left unresolved.
```

**Output Format**:

```markdown
## Open Questions

| ID    | Question                                              | Status         |
| ----- | ----------------------------------------------------- | -------------- |
| OQ-01 | How long should audit logs be retained before purge?  | Open           |
| OQ-02 | Should admins see other users' private notes?         | Deferred to v2 |

### OQ-01: Audit log retention window

**Question**: How long should audit logs (the `audit_events` table recording every incident create/update/delete) be retained before automatic purge?

**Context**: The data model defines `audit_events` with no retention rule, and Constraint CR-04 ("all write actions are logged") produces ~1 row per user action. At current usage (~20k actions/day) the table grows ~7M rows/year.

**Why it matters**: Drives storage cost, query performance on the audit UI, and compliance posture. Too short risks losing evidence for post-incident review; too long drives up DB cost and makes the audit query slow.

**Possible answers**:

- [ ] 30 days — minimal storage (~600k rows), satisfies no external compliance regime, loses quarterly review window
- [ ] 90 days — ~1.8M rows, ~12GB/year, meets SOC2 Type II retention expectation, covers a full billing cycle
- [ ] 1 year — ~7M rows, ~45GB/year, required if we pursue HIPAA; needs partitioning to keep queries <500ms
- [ ] Indefinite — unbounded growth; only viable if we ship archival to cold storage (not in current scope)

**Current implicit default**: The TDD does not specify retention, so logs would grow indefinitely — equivalent to the last option without the archival safeguard.

**Status**: Open — needs legal/compliance input
```

### Automatic Execution

Both steps run automatically with a single `/tdd` command:

```
/tdd [feature-name] [--type backend|ui|api|integration]

Execution:
  Step 1: Generate TDD core sections → Write to file
  Step 2: Invoke Sequential MCP + ultrathink → Analyze TDD → Append Open Questions
  Result: Complete TDD with deep-analysis-generated questions
```

## Outputs

- **Combined TDD** (default): Data models, UI screens, authorization, acceptance criteria — everything for the feature
- **Backend TDD** (`--type backend`): Data models, actions, policies only
- **UI TDD** (`--type ui`): Routes, screens, components only
- **API TDD** (`--type api`): Endpoints, payloads, authentication, error handling
- **Integration TDD** (`--type integration`): Cross-domain interactions, event flows, external systems
- **TDD Review Report** (`--review`): Analysis of existing TDD for code bloat with lean suggestions
- **Consolidated TDD** (`--consolidate`): Tightened document with OQ answers applied and resolved questions removed

## Examples

### Combined TDD (Default)

```
/tdd incidents --prd @specs/prds/02-incident-management.md

# Generates full TDD: data model, UI screens, authorization, acceptance criteria
```

### Split by Type (Complex Features)

```
/tdd payment-webhook --type api

# Generates API-only TDD when feature warrants splitting
```

### Review Existing TDD

```
/tdd --review @specs/tdds/incidents/01-incidents.md

# Analyzes TDD for code bloat, suggests lean improvements
```

### Consolidate After OQ Discussion

```
/tdd --consolidate @specs/tdds/incidents/01-incidents.md

# Applies OQ answers from conversation, tightens document, verifies thresholds
```

## Post-Creation Workflow

After creating a TDD:

1. **Present for review**: Show the complete TDD to the user
2. **Await approval**: Do NOT start implementation until user approves
3. **Resolve Open Questions**: All OQ items should be addressed before implementation
4. **Update if needed**: Incorporate user feedback into the TDD

**CRITICAL**: Creating a TDD is a specification phase, NOT an implementation trigger. Always read and present the TDD for review before any implementation work begins.

## Consolidation Workflow

After Open Questions are answered in discussion, use `--consolidate` to apply answers and tighten the document.

### Usage

```
/tdd --consolidate @specs/tdds/feature-backend.md
```

### Consolidation Actions

1. **Read Context**: Parse TDD and recent conversation for OQ answers
2. **Apply Answers**: Integrate resolved decisions into relevant sections
    - Update Data Model with clarified constraints
    - Add missing Behavior Specs for resolved edge cases
    - Adjust Acceptance Criteria based on scope decisions
3. **Collapse resolved questions into a Decisions table** (do NOT leave answered questions in the verbose `### OQ-NN` Question / Why it matters / Possible answers / Status format):
    - Move every **resolved** question into a `## ✅ Decisions (Resolved)` table with columns **Decision | Choice | Rationale**. Keep the `OQ-NN` / `FQ-NN` id inline in the Decision cell (e.g. `Retention policy (OQ-01)`) so cross-references elsewhere in the doc still resolve.
    - **Delete** the verbose detail blocks of resolved questions (the table is now the record).
    - Keep only genuinely **open** questions under `## Open Questions`, retaining their detail blocks. If none remain, write `*All questions resolved and integrated — see ✅ Decisions (Resolved).*`
    - Update any footer/summary line to reference "✅ Decisions (Resolved)" instead of listing resolved OQ ids.
4. **Tighten Document**:
    - Remove redundant prose
    - Consolidate overlapping sections
    - Ensure tables are scannable
5. **Verify Thresholds**: Check document stays under line limits
6. **Present Changes**: Show summary of modifications

### Before/After Example

**Before consolidation:**

```markdown
## Open Questions

| ID    | Question                          | Status |
|-------|-----------------------------------|--------|
| OQ-01 | Retention policy for audit logs?  | Open   |
| OQ-02 | Max file upload size?             | Open   |
```

**After consolidation** (user answered: 90 days retention, 10MB max):

```markdown
## Data Model

| Attribute     | Type    | Constraints          |
|---------------|---------|----------------------|
| file_size     | integer | max: 10MB            |  <!-- Applied from OQ-02 -->
| ...           | ...     | ...                  |

## Constraints & Rules

| ID    | Rule                                      |
|-------|-------------------------------------------|
| CR-05 | Audit logs retained for 90 days           |  <!-- Applied from OQ-01 -->

## ✅ Decisions (Resolved)

| Decision | Choice | Rationale |
| --- | --- | --- |
| Audit-log retention (OQ-01) | 90 days | Compliance window; bounds storage growth |
| Max file upload size (OQ-02) | 10MB | Covers expected documents; caps abuse |

## Open Questions

*All questions resolved and integrated — see ✅ Decisions (Resolved).*
```

### When to Consolidate

- After answering 3+ Open Questions in discussion
- Before starting implementation
- When TDD context feels bloated
- Before sharing TDD with stakeholders

## Complexity Management

### When to Split TDDs

If a TDD exceeds these thresholds, suggest splitting into multiple documents:

| Indicator                | Threshold              | Action                      |
| ------------------------ | ---------------------- | --------------------------- |
| Data models              | > 3 distinct resources | Split by domain             |
| Acceptance criteria      | > 25 checkboxes        | Split by feature area       |
| Behavior specs           | > 10 scenarios         | Split by user journey       |
| Estimated implementation | > 1 week               | Split by deliverable        |
| Document length          | > 500 lines            | Review for bloat            |
| Document length          | > 1500 lines           | **Hard limit** - must split |

### Master TDD Pattern

For complex features, create a document hierarchy:

1. **Master TDD** (`00-feature-master.md`): Overview, scope, document hierarchy, cross-cutting concerns
2. **Child TDDs** by domain:
    - `01-feature-backend.md` - Data models, actions, policies
    - `02-feature-ui.md` - Routes, screens, components
    - `03-feature-api.md` - Endpoints, webhooks, integrations
    - `04-feature-integration.md` - External systems, event flows

Each child TDD links back to the master and to sibling TDDs.

## Quality Checklist

Before finalizing a TDD, verify:

- [ ] No code blocks longer than 5 lines
- [ ] All requirements traceable to PRD
- [ ] Data model uses tables, not code
- [ ] Interface shows signatures only
- [ ] Acceptance criteria are testable checkboxes
- [ ] **Testing subsection includes coverage targets and test types**
- [ ] **Policy/authentication tests explicitly included for backend/API TDDs**
- [ ] **Accessibility tests explicitly included for UI TDDs**
- [ ] **UI TDDs require end-to-end tests covering render, every change event, every submit success path, and every submit failure branch**
- [ ] **UI acceptance criteria assert backend effect (persisted row, job enqueued, API call) on success — not just redirect or flash**
- [ ] **UI acceptance criteria assert rendered error messages or field errors on failure — not just page chrome (title, header text)**
- [ ] **Bug-fix acceptance criteria require a regression test that fails on the buggy code and passes on the fix**
- [ ] **Every third-party library/SDK/API/CLI/service referenced in the TDD appears in an `External Dependencies` table with name, version, doc URL, fetch date, and section/method/endpoint consulted**
- [ ] **Docs for every third-party surface were fetched fresh via context7 (preferred) or WebFetch during this TDD pass — not recalled from memory, even for well-known vendors (React, Stripe, AWS, etc.)**
- [ ] **Every method name, endpoint path, payload field, header, OAuth scope, error code, and quota stated in the TDD is traceable to an `External Dependencies` row**
- [ ] **Testing subsection requires contract tests against the current vendor contract (recorded fixtures dated within the freshness window, or live contract tests in CI) — not against assumed shapes**
- [ ] No "implementation details" or "how to implement" sections
- [ ] Authorization uses three-layer model (Action, Data, Conditions as needed)
- [ ] Action permissions cover domain actions beyond CRUD where applicable
- [ ] Data permissions specify field visibility per role (when roles see different data)
- [ ] Permission conditions have rule IDs traceable to acceptance criteria
- [ ] **Permission Matrix Test Plan section is present whenever the Authorization section is non-empty**
- [ ] **Test plan row count ≥ `N = A + S + H + R + 2·P + E`** (Layer 1 cells `A`, cross-scope `S`, Hidden pairs `H`, read-only pairs `R`, PC-xx rules `P` counted twice, auth-boundary entry points `E`) — count is asserted in the TDD itself, not left to the reader
- [ ] **Every `❌` cell in the Layer 1 matrix appears as a `Deny` row in the Permission Matrix Test Plan**
- [ ] **Every scope-qualified `✅ Own / Team / Dept` cell has a paired `Cross-scope deny` row**
- [ ] **Every `PC-xx` rule appears in the test plan twice: once with the condition met (Allow), once with the condition not met (Deny)**
- [ ] **Every `Hidden` field-role pair has a row asserting the field is absent from the response payload (and DOM for UI)**
- [ ] **Every `R` read-only field-role pair has a row asserting write attempts are rejected (not silently dropped)**
- [ ] **Every protected entry point has a `401 Unauthenticated` row distinct from `403 Forbidden` rows**
- [ ] **Every Deny row specifies the exact rejection signal expected (e.g. `403 Forbidden`, `{:error, :forbidden}`, `Pundit::NotAuthorizedError`) — never "empty result" or "no-op"**
- [ ] **Every Deny row additionally asserts no state mutation (row unchanged, no audit event, no side effect)**
- [ ] **Any use of `404` as a deny signal is explicitly labeled `Anti-discovery deny` in the row's type column with a justification**
- [ ] **Acceptance Criteria > Testing contains the measurable test-count assertion (`Policy test count ≥ N = …`) referencing the Permission Matrix Test Plan**
- [ ] Behavior specs use Given/When/Then
- [ ] Open Questions generated via Sequential MCP + ultrathink (Phase 2 complete)
- [ ] Each OQ has ID, rationale, possible answers, and status
- [ ] **Each OQ is self-contained: acronyms spelled out, every rule/PRD/attribute reference carries a 1-line inline context, trade-offs are concrete (cost/latency/UX/compliance, not labels like "standard practice"), and the current implicit default is stated — reviewer can answer without re-reading the TDD**
- [ ] TDD presented for user review before implementation
- [ ] Large TDDs split with master document pattern

## Boundaries

**Will:**

- **Always ask clarifying questions before generating** (unless `--no-questions`)
- Generate lean TDDs focused on requirements and contracts
- Use tables and structured formats over prose
- Create testable acceptance criteria
- Reference existing patterns in the codebase
- Identify and suggest removal of code bloat in reviews
- Consolidate TDDs by applying OQ answers and tightening documents

**Will Not:**

- Include full implementation code
- Write function bodies or algorithm details
- Duplicate information that belongs in code
- Generate test implementations
- Create boilerplate or scaffolding code
