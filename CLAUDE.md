# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

claude-code-vitamins is a Claude Code plugin marketplace containing reusable skills and commands for Claude Code. The primary plugin is `specs-plugin` which provides `/prd` and `/tdd` commands for generating Product Requirements Documents and Technical Design Documents.

## Repository Structure

```
.claude-plugin/
  marketplace.json          # Marketplace manifest listing available plugins

plugins/
  specs-plugin/
    .claude-plugin/
      plugin.json           # Plugin manifest
    commands/
      prd.md                # /prd command wrapper
      tdd.md                # /tdd command wrapper
    skills/
      prd-writer/
        SKILL.md            # PRD skill definition
        README.md           # PRD skill documentation
        style-guide.md      # PRD writing conventions
        templates/          # PRD templates (master, feature)
        examples/           # Reference PRD examples
      tdd-writer/
        SKILL.md            # TDD skill definition
        README.md           # TDD skill documentation
        style-guide.md      # TDD writing conventions
        templates/          # TDD templates (backend, ui, api, integration)
        examples/           # Reference TDD examples
```

## Plugin Architecture

### Skill Definition Files (SKILL.md)

Skills are defined with YAML frontmatter:

```yaml
---
name: skill-name
description: What the skill does
allowed-tools: Read, Grep, Glob, Write, Edit, AskUserQuestion, TodoWrite, Task
---
```

The body contains:

- When to use the skill
- Usage syntax with flags
- Key phases and actions
- Include/exclude guidance
- Quality checklists
- Boundaries (will/will not do)

### Command Wrappers (commands/\*.md)

Thin wrappers that invoke skills with arguments:

```markdown
# /command-name Command

## Usage
/command-name [args] [--flags]

## Instructions
Run the `skill-name` skill with the provided arguments.
```

### Templates (templates/\*.md)

Structured templates for generated documents with placeholder sections and consistent formatting patterns.

## Key Design Patterns

### PRD vs TDD Separation

- **PRD**: Defines WHAT problem and WHY. No implementation details.
- **TDD**: Specifies WHAT to build. No code beyond signatures.

### Two-Step Generation Process

Both skills use a two-phase approach:

1. Generate core document sections
2. Invoke Sequential MCP with `--ultrathink` to generate Open Questions via deep analysis

### Output Conventions

PRDs: `specs/prds/{product}/{nn}-{product}-{type}.md` TDDs: `specs/tdds/{feature}/{nn}-{feature}-{type}.md`

Numbering:

- `00-` Master document (hierarchy/overview)
- `01+` Child documents (ordered by priority/type)

### Document Complexity Thresholds

PRDs split when: >8 workflows, >30 requirements, >4 personas, >1000 lines TDDs split when: >3 resources, >25 acceptance criteria, >10 scenarios, >1500 lines

## Common Development Tasks

### Adding a New Skill

1. Create `plugins/{plugin-name}/skills/{skill-name}/` directory
2. Add `SKILL.md` with frontmatter and skill definition
3. Add `style-guide.md` with writing conventions
4. Add `templates/` directory with document templates
5. Add `examples/` directory with reference examples
6. Create command wrapper in `plugins/{plugin-name}/commands/{command}.md`
7. Update `plugins/{plugin-name}/.claude-plugin/plugin.json`

### Adding a New Plugin

1. Create `plugins/{plugin-name}/` directory structure
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills and commands as needed
4. Register in `.claude-plugin/marketplace.json`

## Style Guide Principles

### PRD Style

- WHAT and WHY, never HOW
- Clear over comprehensive
- Measurable over vague
- User-centric always
- Tables for requirements, goals, users
- User workflows over user stories

### TDD Style

- WHAT, not HOW
- Tables over prose
- Signatures only (no function bodies)
- Maximum 5 lines per code block
- Given/When/Then for behavior specs
- Testable checkboxes for acceptance criteria
