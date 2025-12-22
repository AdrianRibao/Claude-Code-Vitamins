# Claude Code Vitamins

A plugin marketplace for [Claude Code](https://claude.ai/code) containing reusable skills and commands to enhance your development workflow.

## Available Plugins

### specs-plugin

Generate excellent Product Requirements Documents (PRDs) and Technical Design Documents (TDDs) with structured workflows and deep analysis.

| Command | Skill      | Purpose                                                                   |
| ------- | ---------- | ------------------------------------------------------------------------- |
| `/prd`  | prd-writer | Generate PRDs focused on problems, goals, users, and success metrics      |
| `/tdd`  | tdd-writer | Generate TDDs focused on requirements, contracts, and acceptance criteria |

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
/prd --help
/tdd --help
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
/prd time-tracking --type master

# Create a feature PRD
/prd time-tracking-mobile --type feature

# Review existing PRD for scope creep
/prd --review @specs/prds/time-tracking/00-master.md

# Consolidate after answering Open Questions
/prd --consolidate @specs/prds/time-tracking/01-mobile.md
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
/tdd incidents --type backend --prd @specs/prds/02-incident-management.md

# Create a UI TDD
/tdd incidents-ui --type ui --prd @specs/prds/02-incident-management.md

# Review existing TDD for bloat
/tdd --review @specs/tdds/incidents/01-incident-resource.md

# Consolidate after answering Open Questions
/tdd --consolidate @specs/tdds/incidents/01-incident-resource.md
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

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. CREATE PRD                                                  │
│     /prd feature --type master                                  │
│                                                                 │
│     • Define problem, goals, users, requirements                │
│     • Generate Open Questions via deep analysis                 │
│     • Review and consolidate                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  2. CREATE TDD                                                  │
│     /tdd feature --type backend --prd @specs/prds/feature.md    │
│                                                                 │
│     • Define data models, contracts, acceptance criteria        │
│     • Generate Open Questions via deep analysis                 │
│     • Review and consolidate                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  3. IMPLEMENT                                                   │
│                                                                 │
│     • PRD and TDD are complete and approved                     │
│     • Ready for implementation                                  │
└─────────────────────────────────────────────────────────────────┘
```

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
