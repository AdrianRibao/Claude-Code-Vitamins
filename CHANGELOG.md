# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- **tdd-writer, prd-writer**: `✅ Decisions (Resolved)` is now one `### D-NN — title` heading per decision with a `**Choice.**` line and a `**Why.**` line, instead of a Decision / Choice / Rationale table. Rationales with real trade-offs made the table wider than a terminal and unreadable in review, and a paragraph form gets joined into one line by `wrap = "no"` formatters; headings keep each idea on its own line and stay jumpable. Applied to the Phase 2 output format, the consolidation workflow and examples, the templates (combined/backend TDD, feature/master PRD), the style guides and the quality checklists. **task-writer**: the struck-through bullet form (`~~question~~ **Decided …**`) is replaced by the same headings (`### D-NN — question` with `**Decided YYYY-MM-DD.**` and `**Why.**` lines; open forks as `### OD-NN — question (open)` with `**Choices.**` and `**Recommended.**`), in the skill, style guide, template and both examples; reformatting legacy entries is preserved-text-only. **bugfix-writer**: `--consolidate` records answered `OD-NN` items under `## ✅ Decisions (Resolved)` in the same heading shape (style guide, template, outputs). Bump 1.12.1.

### Added

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
