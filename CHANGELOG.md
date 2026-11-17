# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **qa-runner skill and `/qa` command**: runs a spec's QA plan in a real browser with [agent-browser](https://agent-browser.dev) — the only engine, no fallback: agent-browser missing or `agent-browser doctor --offline` failing (it includes a live launch test) means QA has not run, and only the user can waive the gate. Scripted `QA-N` checks run to the end in one session per role and are judged against their `Passes when`; an exploratory pass uses agent-browser's bundled `dogfood` skill with explicit overrides (vault sessions only, never `state save`, evidence directory, no issue quota). Logins go through the agent-browser auth vault, one profile per role **and environment** (a profile stores one login URL), saved by the user in their own terminal; the runner checks each login landed on the environment's origin. **Defects QA finds on the screens and flows the change touches are fixed, not filed — including pre-existing ones** (re-tested, re-reviewed, re-run; the user is asked first when a fix is large, touches a `Do NOT change` item, or touches data, money or security); pre-existing defects on untouched screens are filed with `/bugfix`, improvements with `/task`. Results include `pass (after fix)`, `pass (baseline approved …)` after the user approves a new visual baseline, `informational` for `vitals` on local, and rule-based `skipped`; an `--only` re-run merges into the record. Local runs gate the commit after the code review; projects with staging run `--env staging` after deploy, before promotion. Evidence goes to git-ignored `specs/qa/runs/`; approved baselines to committed `specs/qa/baselines/`. Its style guide is the single QA plan convention: UI changes only (API and backend work has no browser QA; API bug fixes are still confirmed at the live endpoint), environments, per-environment logins, the closed capability list (`ui-flow`, `dialog`, `network`, `console-clean`, `a11y`, `responsive`, `visual`, `vitals`), pass conditions, evidence and results; specs say WHAT, never an agent-browser command.

- **bugfix-writer, task-writer, tdd-writer**: UI bugs, UI tasks and TDDs with UI (any type, including the default combined) carry a `QA plan` section (template + style guide + examples); non-UI work records `QA: n/a — no UI change`. UI bugs are reproduced with agent-browser. The bugfix *Confirmed fixed in the app* and task *Verified in the running system* gates now mean a passing `/qa` run for UI changes; QA cannot be waived except by the user; the scope gates allow defects QA found on what the change touches to be fixed in it; `--resolve` and `--ship` run `/qa` when a UI change has no local QA record; hand-offs stage the QA record, `.gitignore` line and baselines, and remind the user about staging. claude-in-chrome references are removed. TDDs with UI add the QA criteria to `Acceptance Criteria > Testing` (staging only when the plan has a staging row) and derive their checks from the PRD's user workflows. **ac-checker** reports the QA criteria as `Manual`. PRDs carry no QA plan. Bump 1.14.0.

- **tdd-writer**: Permission Matrix formula extended with the `F` (referent) term — `N = A + S + F + H + R + 2·P + E` — closing the IDOR / confused-deputy gap: one `Referent deny` row per (action, foreign-key input) pair whose referenced record is scope-qualified (authorized actor inside their own scope submits another scope's id → explicit rejection, zero state written)

    - New `Referent deny` type in the closed Test Plan `Type` enum, with worked example rows in the style guide and the combined/backend/api templates
    - New **referent sweep test** requirement when `F > 0`: a code-derived test that introspects schema/resource definitions, enumerates every (action, FK-input) pair, and asserts each declares an ownership/scope validation (framework pointers for Ash, Rails, Django, Prisma/TypeORM)
    - Global, non-scoped lookup tables (currencies, countries) are exempt from `F`

- **ac-checker**: Derives referent pairs from the Interface Contract / Data Model, requires a `Referent deny` row per pair (Critical when missing), verifies the referent sweep test exists, and accepts the legacy formula on pre-`F` TDDs with a Medium finding

- **ac-checker skill**: New skill to verify acceptance criteria implementation status

    - Searches test files for matching test cases for each criterion
    - Searches source files for implementation code
    - Validates test coverage meets TDD targets (≥80%, ≥90%, ≥95%)
    - Compares checkbox completion status with actual implementation
    - Generates detailed implementation reports
    - Optional `--update` flag to auto-mark completed criteria in TDD
    - Optional `--coverage` flag to run coverage analysis
    - Optional `--branch` flag to compare against specific branch
    - Command: `/specs-plugin:ac-checker [tdd-path] [--flags]`

- mdformat compatibility guide in CLAUDE.md documenting common issues and solutions

- Comprehensive README.md documentation for ac-checker skill

- Updated workflow diagram to include verification step

### Changed

- **bugfix-writer, task-writer, tdd-writer**: the `code-review` skill now runs on the final diff after implementation and before any commit. **bugfix-writer** adds an eighth gate, *Code reviewed* (Phase 4 step between the green suite and the running-app check; `Verification > Code review` in the template; a mandatory checklist criterion; `--resolve` runs the review when none is recorded). **task-writer** adds a sixth ship gate, *Code reviewed* (Phase 5 step; a `Code review` line in Verification, `n/a — no code changed` for operational work; `--ship` runs the review when none is recorded). Both triage every finding instead of applying it wholesale: fix it in scope and re-run the tests, file it with `/bugfix` or `/task` when out of scope, or dismiss it with a reason; a review that cannot run falls back to an independent `Task` subagent and is never skipped silently. **tdd-writer** writes two code-review criteria at the end of every template's `Acceptance Criteria > Code Quality` (the TDD writer does not implement, so the criterion carries the rule to the implementer) and documents the order tests -> `ac-checker` -> `code-review` -> commit. **ac-checker** reports those criteria as `Manual` process criteria rather than missing code. README diagrams, style guides, templates and examples updated. Bump 1.13.0.

- **tdd-writer, prd-writer**: `✅ Decisions (Resolved)` is now one `### D-NN — title` heading per decision with a `**Choice.**` line and a `**Why.**` line, instead of a Decision / Choice / Rationale table. Rationales with real trade-offs made the table wider than a terminal and unreadable in review, and a paragraph form gets joined into one line by `wrap = "no"` formatters; headings keep each idea on its own line and stay jumpable. Applied to the Phase 2 output format, the consolidation workflow and examples, the templates (combined/backend TDD, feature/master PRD), the style guides and the quality checklists. **task-writer**: the struck-through bullet form (`~~question~~ **Decided …**`) is replaced by the same headings (`### D-NN — question` with `**Decided YYYY-MM-DD.**` and `**Why.**` lines; open forks as `### OD-NN — question (open)` with `**Choices.**` and `**Recommended.**`), in the skill, style guide, template and both examples; reformatting legacy entries is preserved-text-only. **bugfix-writer**: `--consolidate` records answered `OD-NN` items under `## ✅ Decisions (Resolved)` in the same heading shape (style guide, template, outputs). Bump 1.12.1.

- Bumped specs-plugin to version 1.10.0

- Updated specs-plugin to version 1.1.0

- Enhanced CLAUDE.md with mdformat validation patterns

- Improved project workflow to include implementation verification phase

## [1.0.0] - 2026-01-10

### Added

- **prd-writer skill**: Generate Product Requirements Documents
    - Master, feature, API, and integration PRD types
    - Focus on problems, goals, users, and success metrics
    - Open Questions generation via Sequential MCP + ultrathink
    - Review mode for scope creep analysis
    - Consolidate mode for applying OQ answers
    - Command: `/specs-plugin:prd [product] --type [master|feature|api|integration]`
- **tdd-writer skill**: Generate Technical Design Documents
    - Backend, UI, API, and integration TDD types
    - Focus on requirements, contracts, and acceptance criteria
    - Structured acceptance criteria with 5 required subsections
    - Comprehensive testing requirements and coverage targets
    - UI tests section for accessibility and keyboard navigation
    - Code Quality section for linting, formatting, and security
    - Open Questions generation via Sequential MCP + ultrathink
    - Review mode for bloat analysis
    - Consolidate mode for applying OQ answers
    - Command: `/specs-plugin:tdd [feature] --type [backend|ui|api|integration]`
- MCP server integrations:
    - Context7 integration for library documentation lookups
    - Sequential Thinking integration for deep Open Questions analysis
- Comprehensive style guides for both PRD and TDD writing
- Template library with master and feature templates
- Example PRDs and TDDs for reference
- Project documentation in CLAUDE.md for Claude Code guidance
- Marketplace manifest for plugin distribution

### Changed

- Updated Context7 MCP to use official Upstash package
- Enhanced skill frontmatter with clear allowed-tools lists
- Improved TDD templates with testing requirements

## [0.1.0] - Initial Release

### Added

- Initial project structure
- Marketplace configuration
- specs-plugin foundation
- Basic PRD and TDD generation capabilities

[0.1.0]: https://github.com/AdrianRibao/Claude-Code-Vitamins/releases/tag/v0.1.0
[1.0.0]: https://github.com/AdrianRibao/Claude-Code-Vitamins/releases/tag/v1.0.0
[unreleased]: https://github.com/AdrianRibao/Claude-Code-Vitamins/compare/v1.0.0...HEAD
