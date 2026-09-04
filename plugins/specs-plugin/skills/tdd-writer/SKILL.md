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
/tdd [feature-name] [--prd @path] [--no-questions] [--ask] [--review] [--consolidate @path] [--type backend|ui|api|integration]
```

| Flag                  | Purpose                                             |
| --------------------- | --------------------------------------------------- |
| `--prd @path`         | Reference PRD for requirements                      |
| `--no-questions`      | Skip upfront questions (use with comprehensive PRD) |
| `--ask`               | Surface decide-and-record items as Open Questions   |
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
8. **Write TDD**: Save to file with placeholder `✅ Decisions (Resolved)` and Open Questions sections and a populated `External Dependencies` table

### Phase 2: Decision Triage (decide by default)

1. **Invoke Sequential MCP**: Use `--ultrathink` to scan the complete TDD for ambiguities, missing decisions, edge cases, authorization gaps, and scope gaps
2. **Classify each finding** (see [Question Policy](#question-policy)): decide-and-record, ask, or non-issue
3. **Decide-and-record**: Pick the answer you are confident in, fold it into the relevant section (data model, contract, authorization, behavior spec, AC), and log it under `## ✅ Decisions (Resolved)` with a one-line rationale
4. **Ask sparingly**: Only findings a human must settle become `OQ-NN` entries under `## Open Questions`. If none survive, write the no-open-questions line

### Phase 3: Finalization

1. **Update References**: Update master TDDs and related documents to reference the new TDD
2. **Validate**: Review against PRD, check completeness, verify testability
3. **Quality Check**: Ensure no code blocks > 5 lines, all criteria testable, all links valid
4. **Present for Review**: Show complete TDD to user, await approval before implementation

## Include in TDDs

| Element                     | Purpose                                                                                                                                          | Example                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| Requirements                | WHAT needs to be built                                                                                                                           | "Users can create incidents"                                 |
| Data Models                 | Attributes, types, constraints (tables)                                                                                                          | `hours_adjustment: decimal, required`                        |
| Interface Contracts         | Function signatures, action names                                                                                                                | `create_incident(params, actor:)`                            |
| Behavior Specs              | Input → Output expectations                                                                                                                      | "Creating incident sends notification"                       |
| Acceptance Criteria         | How to verify completion                                                                                                                         | "[ ] Employee can create incident"                           |
| Testing Requirements        | Test types, coverage targets                                                                                                                     | "[ ] Policy tests verify all auth rules"                     |
| Constraints & Rules         | Business logic boundaries                                                                                                                        | "DL-01: Hours adjustment model"                              |
| Authorization               | Three-layer permission model (tables)                                                                                                            | Action, Data, and Condition permissions                      |
| Permission Matrix Test Plan | Enumerated PT-xx rows: every cell, every Hidden/R field-role pair, every PC-xx (met + unmet), cross-scope denies, referent denies, auth boundary | "PT-07: Layer 1, Owner × approve, Deny, expects 403"         |
| Related Documents           | Links to PRDs, master TDDs, siblings                                                                                                             | "Parent PRD: [link], Backend: [link]"                        |
| External Dependencies       | Every third-party library/SDK/API/CLI used, with version, doc URL, fetch date, section consulted                                                 | "stripe-node v17.4.0, /payment-intents, 2026-05-26"          |
| Decisions                   | Judgment calls you made, with rationale                                                                                                          | "D-01: Soft-delete via `deleted_at`, 30-day purge"           |
| Open Questions              | Only what a human must decide (often empty)                                                                                                      | "OQ-01: Is 1-year audit retention a compliance requirement?" |

### Authorization Model (Three Layers)

TDDs specify authorization through three complementary layers. Not every TDD needs all three — use what fits the feature's complexity. If a feature has no authentication (public pages, anonymous endpoints), skip the authorization section entirely.

**Layer 1 - Action Permissions**: Who can perform which actions, and on what scope. Expand beyond basic CRUD to include domain-specific actions (approve, publish, escalate, export, delegate). Use scope qualifiers: Own, Team, Department, Org, All.

**Layer 2 - Data Permissions**: Which fields each role can see or modify. Use visibility levels: **RW** (read-write), **R** (read-only), **Hidden** (not visible). Only needed when different roles see different fields or have field-level edit restrictions.

**Layer 3 - Permission Conditions**: When permissions apply or are revoked. Captures status-based, time-based, relationship-based, and approval-based constraints. Use rule IDs (PC-01, PC-02) for traceability to acceptance criteria.

See [style-guide.md](style-guide.md) for full format reference and examples.

### Permission Matrix Test Coverage Requirements (non-negotiable)

**The permission matrix is the highest-risk surface in the system. A green test suite that does not exercise every deny path is worse than no tests — it grants false confidence to a reviewer who is about to merge a privilege-escalation bug.** Every TDD whose Authorization section is non-empty MUST include a **Permission Matrix Test Plan** that enumerates one row per required test, and the Acceptance Criteria MUST require that count to be met. Deny coverage is the load-bearing requirement — allow paths are easy to write by accident while shipping a feature; deny paths are not.

**Minimum required test count (the formula)**:

Total required policy tests `N = A + S + F + H + R + 2·P + E`, where:

- `A` = number of cells in the Layer 1 actor × action matrix (one test per cell — `✅` cells assert the action succeeds within scope, `❌` cells assert explicit Forbidden, never silent failure or no-op).
- `S` = number of scope-qualified `✅ Own / Team / Dept` cells in Layer 1 (one paired cross-scope deny test each: actor attempts the same action on a record **outside** their scope; expected Forbidden).
- `F` = number of **referent inputs** — every (action, foreign-key input) pair where the action accepts an attribute or argument that references another **scope-qualified** record (one `Referent deny` test each: an **authorized** actor, acting **inside** their own scope, submits an id belonging to **another** scope; expected explicit rejection AND zero state written). `F` counts *inputs*, not actions — an action accepting three FKs contributes 3. It covers create and update payloads alike, plus nested/relationship writes, bulk id lists, and polymorphic ids. References to global, non-scoped lookup tables (e.g. a currency or country list) are exempt.
- `H` = number of `(role, field)` pairs in Layer 2 marked `Hidden` (one test each asserting the field is absent from the response payload — and absent from the DOM for UI surfaces — for that role).
- `R` = number of `(role, field)` pairs in Layer 2 marked `R` read-only (one test each asserting a write attempt is rejected with 422 / changeset error / Forbidden — never silently dropped).
- `P` = number of Layer 3 `PC-xx` permission condition rules (two tests each: one with the condition **met** so the action is allowed, one with the condition **not met** so the action is denied with explicit Forbidden).
- `E` = number of protected entry points (one test each asserting unauthenticated request returns 401 — not 403, not 404; the distinction is part of the contract).

**What counts as a deny test (strict)**:

| Counts                                                                                                                                     | Does NOT count                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Asserts explicit `403 Forbidden` / `{:error, :forbidden}` / `Pundit::NotAuthorizedError`                                                   | Asserts only that result list is empty (could be empty for many reasons unrelated to authorization)                                                                  |
| Asserts the unauthorized action did **not** mutate state (row unchanged, no audit event)                                                   | Asserts only the HTTP status without checking the side effect didn't happen                                                                                          |
| Asserts the response payload omits the Hidden field                                                                                        | Asserts only that the response was 200 (a 200 with the field still leaked is the exact bug we're guarding against)                                                   |
| `404 Not Found` only when explicitly labeled `anti-discovery` in the test plan                                                             | `404` used as a generic deny — it hides the real authorization decision and breaks audit/forensic analysis                                                           |
| Test fixture uses a real authenticated actor in the **denied** role                                                                        | Test that disables auth, stubs the policy, or runs as a superuser to "simplify setup"                                                                                |
| Referent deny: authorized actor's payload names another scope's record; asserts explicit rejection AND no row/link written in either scope | Trusting that "the FK exists and the policy passed" — both pass in the IDOR case, because the actor and tenant are legitimate; only the referenced record is foreign |

**Why `F` exists (the IDOR / confused-deputy gap)**: the Layer 1 matrix varies *who* acts (subject) and *whose record* is acted on (object), but says nothing about *which other records the payload points at* (referent). A fully green matrix without `F` rows still ships the classic IDOR: an authorized actor, inside their own scope, submits another scope's id as an argument and the write persists a cross-scope link. Every `F` row holds the actor and the object legitimate and varies only the referenced id.

**Referent sweep test (required alongside the `F` rows)**: `F` rows are enumerated by hand — by the same author who may have forgotten the validation. Any TDD introducing actions that accept references to other scope-qualified records MUST also require a **referent sweep test**: a single test that introspects the application's schema/resource definitions, enumerates every (action, foreign-key input) pair, and asserts each one declares an ownership/scope validation. It must fail for a *newly added* action that omits the check, without anyone editing the test. The sweep proves nothing is **forgotten**; the `F` rows prove the check actually **rejects**. Neither is sufficient alone. Enumeration sources per stack (non-exhaustive):

| Stack            | Enumerate (action, FK-input) pairs from                                     |
| ---------------- | --------------------------------------------------------------------------- |
| Ash / Elixir     | `Ash.Resource.Info.actions/1` + each action's accepted attributes/arguments |
| Rails            | `reflect_on_all_associations` + strong-params permit lists                  |
| Django           | model `_meta.get_fields()` + serializer/form field sets                     |
| Prisma / TypeORM | schema metadata + DTO validation decorators                                 |

**Permission Matrix Test Plan (required artifact)**: every TDD must include this section, generated mechanically from the three layer tables. See [style-guide.md](style-guide.md#permission-matrix-test-plan) for the exact format.

**Acceptance gate**: a TDD is NOT acceptable for implementation until:

1. The Permission Matrix Test Plan section exists and has `≥ formula` rows.
2. Every row has a unique `PT-xx` ID, an explicit `Allow` / `Deny` / `Cross-scope deny` / `Referent deny` / `Anti-discovery deny` type label, and an explicit expected response.
3. Every `❌` cell in Layer 1 is present as a `Deny` row in the plan.
4. Every `PC-xx` rule appears twice in the plan (one met, one unmet).
5. Every `Hidden` and `R` field-role pair appears in the plan.
6. Every (action, foreign-key input) pair whose referenced record is scope-qualified appears as a `Referent deny` row.
7. When `F > 0`, the Acceptance Criteria > Testing subsection requires the referent sweep test (code-derived enumeration asserting every such pair declares an ownership/scope validation).
8. The Acceptance Criteria > Testing subsection contains the measurable count assertion (e.g., "Policy test count ≥ N = A + S + F + H + R + 2·P + E" with the arithmetic spelled out).

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

## Decision Triage Workflow

TDD creation follows a **two-step process**: the core document is written first, then a deep-analysis pass hunts for gaps. The pass **decides by default** and only escalates what a human genuinely has to settle.

### Step 1: Create TDD Core

Generate all TDD sections EXCEPT Decisions and Open Questions. For combined TDDs (default), include both backend and UI sections:

1. Overview and Scope
2. Data Model (tables)
3. Interface Contract (signatures only)
4. Authorization (Action Permissions, Data Permissions, Permission Conditions)
5. **Permission Matrix Test Plan** — mechanically enumerate one `PT-xx` row for every cell, cross-scope deny, referent (action, FK-input) pair, Hidden/R field-role pair, PC-xx (met + unmet), and auth boundary. The row count MUST satisfy the `N = A + S + F + H + R + 2·P + E` formula
6. UI: Routes, Screens, Components (if feature has UI)
7. Behavior Specifications (Given/When/Then)
8. Acceptance Criteria (checkboxes with authorization testing — including the measurable test-count assertion referencing the Permission Matrix Test Plan)
9. Related Documents

Write the TDD to file with placeholder sections:

```markdown
## ✅ Decisions (Resolved)

*Generating via deep analysis...*

## Open Questions

*Generating via deep analysis...*
```

### Question Policy

**Default: decide.** The deep-analysis pass exists to find gaps, not to produce a list of questions. Once a gap is found, close it yourself whenever you can do so with confidence — a reviewer's attention is the scarcest resource in this process, and a TDD that arrives with five questions the author could have answered is worse than one that arrives with five recorded decisions. Classify every finding into one of three tiers:

| Tier                  | When                                                                                                                                                                                                                   | Action                                                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Decide-and-record** | A defensible answer follows from the PRD, the codebase's existing patterns, fetched third-party docs, or established engineering practice (architecture, naming, validation limits, indexes, error handling, defaults) | Choose it, fold it into the relevant section, add a row to `## ✅ Decisions (Resolved)` (Decision / Choice / Rationale) |
| **Ask**               | Only a human can settle it: business policy the PRD leaves open, legal/compliance-driven values, infrastructure budget, external contracts or SLAs, a user-facing behavior fork the product owner must call            | Write an `OQ-NN` entry under `## Open Questions` with a recommended option                                              |
| **Non-issue**         | Already specified, or the answer would not change the implementation                                                                                                                                                   | Say nothing; do not manufacture questions                                                                               |

**Ask-tier test** — an Open Question is legitimate only when *all three* hold:

- Answering it changes the implementation, not just the wording of the document
- You could not reach a confident answer from the PRD, the codebase, or fetched docs
- The reviewer has information or authority you do not

**Authorization gaps are always decide-and-record**: fill any missing action permission, data visibility rule, or untested Permission Condition with the least-privilege answer, add the corresponding `PT-xx` rows, and log the decision. Never leave an authorization hole as a question.

**Irreversible-decision guardrail**: when a decision touches money, legal exposure, data retention or privacy, destructive migrations, or a public API contract, ask even if you have a preference — and state that preference as the recommended option.

If no finding survives the Ask-tier test, `## Open Questions` contains exactly this line and nothing else:

```markdown
*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
```

Never pad the section. Three decisions and zero questions is a better outcome than two manufactured questions.

`--ask` overrides the default and surfaces decide-and-record items as Open Questions as well (each still carrying your recommended answer), for the rare TDD where the user wants to review every judgment call before implementation.

### Step 2: Deep Analysis and Triage (ultrathink)

After the TDD core is written, invoke Sequential MCP with `--ultrathink` depth to analyze the complete TDD, resolve what can be resolved, and escalate only what cannot.

**Analysis Focus Areas**:

| Area                     | What to look for                                                         | Usual tier                                              |
| ------------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------- |
| Requirements Ambiguity   | Unclear acceptance criteria, missing edge cases                          | Decide-and-record                                       |
| Stakeholder Input Needed | Business decisions, policy clarifications                                | Ask                                                     |
| Technical Decisions      | Architecture choices, integration approaches                             | Decide-and-record                                       |
| Scope Boundaries         | What's explicitly out of scope, future considerations                    | Decide-and-record                                       |
| Data Constraints         | Validation rules, retention policies, limits                             | Decide-and-record (retention driven by compliance: Ask) |
| Authorization Gaps       | Missing action permissions, unclear data visibility, untested conditions | Always decide-and-record                                |

**Self-contained rule (non-negotiable, applies to both decisions and questions)**:

Every Decision row and every Open Question must be understandable **without re-reading the TDD**. The reviewer should be able to open the section cold and either accept the decision or answer the question. Concretely:

- **No bare acronyms.** Spell them out on first use, even if defined earlier in the TDD (e.g. write "Permission Condition PC-03 (only owners can edit after approval)" not "PC-03").
- **Inline 1-line context for every reference.** Any mention of a PRD ID, Constraint Rule (CR-xx), Permission Condition (PC-xx), Data Rule (DL-xx), Acceptance Criterion, attribute, or sibling TDD must carry a parenthetical summary of what that thing is.
- **Concrete trade-offs.** Rationales and possible answers must describe the real consequence of each choice (cost, latency, UX impact, compliance risk) — not just the value. "90 days — meets SOC2 retention but adds ~40GB/month storage" beats "90 days (standard practice)".
- **State the default the TDD currently implies**, so the reviewer knows what happens if they do nothing.

**Sequential MCP Prompt Template**:

```
Analyze this TDD for ambiguities, missing decisions, edge cases,
authorization gaps and scope gaps that would affect implementation.

For EACH finding, first try to resolve it yourself:

- If a defensible answer follows from the PRD, the codebase's existing
  patterns, fetched third-party docs, or established engineering
  practice, DECIDE. Output it as a decision: id (D-01, D-02, ...), the
  choice, a one-line rationale with concrete trade-offs, and the TDD
  section to update. Authorization gaps are ALWAYS decided, using the
  least-privilege answer.
- Only if a human must settle it (business policy the PRD leaves open,
  legal or compliance values, infrastructure budget, external contracts,
  or a user-facing behavior fork the product owner must call) output it
  as an open question: id (OQ-01, ...), a self-contained question,
  1-line context, why it matters, possible answers with concrete
  trade-offs, the current implicit default, and the recommended option.
- Drop everything else.

Each decision and question MUST be self-contained — spell out every
acronym on first use, add a 1-line parenthetical for every referenced
rule, attribute, PRD section or sibling TDD, and describe trade-offs
concretely (cost, latency, UX, compliance).

Expect zero or very few open questions. Do not manufacture questions
to fill the section.
```

**Output Format**:

```markdown
## ✅ Decisions (Resolved)

| Decision                        | Choice                                        | Rationale                                                                                                           |
| ------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Incident deletion (D-01)        | Soft-delete via `deleted_at`, purge after 30d | Constraint CR-04 (all writes are audited) needs the row for the audit trail; hard delete would orphan audit events |
| Viewer access to notes (D-02)   | Viewers cannot read `private_notes`           | Least privilege; the PRD grants viewers read-only *summary* access, and PT-14/PT-15 now assert the deny            |

## Open Questions

| ID    | Question                                              | Status |
| ----- | ----------------------------------------------------- | ------ |
| OQ-01 | Is 1-year audit log retention a compliance requirement? | Open   |

### OQ-01: Audit log retention window

**Question**: How long should audit logs (the `audit_events` table recording every incident create/update/delete) be retained before automatic purge?

**Context**: The data model defines `audit_events` with no retention rule, and Constraint CR-04 ("all write actions are logged") produces ~1 row per user action. At current usage (~20k actions/day) the table grows ~7M rows/year.

**Why it matters**: Drives storage cost, query performance on the audit UI, and compliance posture. This is a compliance value the PRD does not state, so it cannot be decided from the codebase.

**Possible answers**:

- [ ] 90 days — ~1.8M rows, ~12GB/year, meets SOC2 Type II retention expectation, covers a full billing cycle **(recommended unless HIPAA is on the roadmap)**
- [ ] 1 year — ~7M rows, ~45GB/year, required if we pursue HIPAA; needs partitioning to keep queries <500ms
- [ ] Indefinite — unbounded growth; only viable if we ship archival to cold storage (not in current scope)

**Current implicit default**: The TDD does not specify retention, so logs would grow indefinitely — equivalent to the last option without the archival safeguard.

**Status**: Open — needs legal/compliance input
```

When nothing survives the Ask-tier test:

```markdown
## Open Questions

*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
```

### Automatic Execution

Both steps run automatically with a single `/tdd` command:

```
/tdd [feature-name] [--type backend|ui|api|integration]

Execution:
  Step 1: Generate TDD core sections → Write to file
  Step 2: Invoke Sequential MCP + ultrathink → Triage → Record decisions, append only real Open Questions
  Result: Complete TDD with decisions recorded and zero-or-few questions for the reviewer
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
3. **Review decisions, resolve questions**: Skim `✅ Decisions (Resolved)` and override any you disagree with; answer every remaining OQ item before implementation
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
    - Move every **resolved** question into the `## ✅ Decisions (Resolved)` table (extend the existing one from creation time; create it if absent) with columns **Decision | Choice | Rationale**. Keep the `OQ-NN` / `FQ-NN` id inline in the Decision cell (e.g. `Retention policy (OQ-01)`) so cross-references elsewhere in the doc still resolve.
    - **Delete** the verbose detail blocks of resolved questions (the table is now the record).
    - Keep only genuinely **open** questions under `## Open Questions`, retaining their detail blocks. If none remain, write `*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*`
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

*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
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
- [ ] **Test plan row count ≥ `N = A + S + F + H + R + 2·P + E`** (Layer 1 cells `A`, cross-scope `S`, referent-input pairs `F`, Hidden pairs `H`, read-only pairs `R`, PC-xx rules `P` counted twice, auth-boundary entry points `E`) — count is asserted in the TDD itself, not left to the reader
- [ ] **Every `❌` cell in the Layer 1 matrix appears as a `Deny` row in the Permission Matrix Test Plan**
- [ ] **Every scope-qualified `✅ Own / Team / Dept` cell has a paired `Cross-scope deny` row**
- [ ] **Every (action, foreign-key input) pair whose referenced record is scope-qualified has a `Referent deny` row — an authorized actor inside their own scope submitting another scope's id is explicitly rejected with zero state written**
- [ ] **When actions accept references to other scope-qualified records, the Testing subsection requires a referent sweep test — a code-derived test that enumerates every (action, FK-input) pair from the schema/resource definitions and asserts each declares an ownership/scope validation**
- [ ] **Every `PC-xx` rule appears in the test plan twice: once with the condition met (Allow), once with the condition not met (Deny)**
- [ ] **Every `Hidden` field-role pair has a row asserting the field is absent from the response payload (and DOM for UI)**
- [ ] **Every `R` read-only field-role pair has a row asserting write attempts are rejected (not silently dropped)**
- [ ] **Every protected entry point has a `401 Unauthenticated` row distinct from `403 Forbidden` rows**
- [ ] **Every Deny row specifies the exact rejection signal expected (e.g. `403 Forbidden`, `{:error, :forbidden}`, `Pundit::NotAuthorizedError`) — never "empty result" or "no-op"**
- [ ] **Every Deny row additionally asserts no state mutation (row unchanged, no audit event, no side effect)**
- [ ] **Any use of `404` as a deny signal is explicitly labeled `Anti-discovery deny` in the row's type column with a justification**
- [ ] **Acceptance Criteria > Testing contains the measurable test-count assertion (`Policy test count ≥ N = …`) referencing the Permission Matrix Test Plan**
- [ ] Behavior specs use Given/When/Then
- [ ] Decision triage run via Sequential MCP + ultrathink (Phase 2 complete)
- [ ] Every decide-and-record finding is folded into its section and logged in `✅ Decisions (Resolved)` with a rationale; every authorization gap was decided (least privilege), never asked
- [ ] Every OQ passes the Ask-tier test (changes the implementation, not answerable with confidence, reviewer holds the authority) and carries a recommended option
- [ ] Open Questions is either genuine questions or exactly the no-open-questions line — never padded
- [ ] **Each Decision and OQ is self-contained: acronyms spelled out, every rule/PRD/attribute reference carries a 1-line inline context, trade-offs are concrete (cost/latency/UX/compliance, not labels like "standard practice"), and the current implicit default is stated — reviewer can answer without re-reading the TDD**
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
