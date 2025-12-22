---
name: tdd-writer
description: Generate lean Technical Design Documents focused on requirements, contracts, and acceptance criteria. Use when creating TDDs, converting PRDs to specs, or documenting backend resources, APIs, or UI components.
allowed-tools: Read, Grep, Glob, Write, Edit, AskUserQuestion, TodoWrite, Task
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
/tdd [feature-name] [--type backend|ui|api|integration] [--prd @path] [--no-questions] [--review] [--consolidate @path]
```

| Flag | Purpose |
|------|---------|
| `--type` | TDD type: backend, ui, api, integration |
| `--prd @path` | Reference PRD for requirements |
| `--no-questions` | Skip upfront questions (use with comprehensive PRD) |
| `--review` | Analyze existing TDD for bloat |
| `--consolidate @path` | Apply OQ answers, tighten document |

## Output Location

TDDs are created at:

```
specs/tdds/{feature}/{nn}-{feature}-{type}.md
```

**Examples:**

| Command | Output Path |
|---------|-------------|
| `/tdd user-auth --type backend` | `specs/tdds/user-auth/01-user-auth-backend.md` |
| `/tdd user-auth --type ui` | `specs/tdds/user-auth/02-user-auth-ui.md` |
| Master TDD (complex features) | `specs/tdds/user-auth/00-user-auth-master.md` |

**Numbering convention:**
- `00-` Master TDD (overview, document hierarchy)
- `01-` Backend resource
- `02-` UI/Frontend
- `03-` API
- `04-` Integration

Ask user to confirm or override path during Phase 0 questions.

## Resources

- [style-guide.md](style-guide.md) - TDD writing conventions and rules
- [templates/](templates/) - TDD templates by type (backend, ui, api, integration)
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

5. **Discover**: Read PRD, understand requirements, identify scope boundaries
6. **Map Document Hierarchy**: Identify parent PRD, master TDD, sibling TDDs, and related specs
7. **Analyze Existing Patterns**: Find related TDDs and resources in the codebase for consistency
8. **Structure**: Organize into logical sections following lean TDD template
9. **Specify**: Write requirements, data models, contracts, acceptance criteria in tables
10. **Link Documents**: Add "Related Documents" section with all relevant PRDs and TDDs
11. **Write TDD**: Save to file with placeholder Open Questions section

### Phase 2: Open Questions Deep Analysis

12. **Invoke Sequential MCP**: Use `--ultrathink` for maximum depth analysis
13. **Analyze Gaps**: Identify ambiguities, missing decisions, edge cases, scope boundaries
14. **Generate Questions**: Create structured OQ entries with IDs, rationale, and possible answers
15. **Append to TDD**: Update the Open Questions section with generated content

### Phase 3: Finalization

16. **Update References**: Update master TDDs and related documents to reference the new TDD
17. **Validate**: Review against PRD, check completeness, verify testability
18. **Quality Check**: Ensure no code blocks > 5 lines, all criteria testable, all links valid
19. **Present for Review**: Show complete TDD to user, await approval before implementation

## Include in TDDs

| Element              | Purpose                                 | Example                                |
| -------------------- | --------------------------------------- | -------------------------------------- |
| Requirements         | WHAT needs to be built                  | "Users can create incidents"           |
| Data Models          | Attributes, types, constraints (tables) | `hours_adjustment: decimal, required`  |
| Interface Contracts  | Function signatures, action names       | `create_incident(params, actor:)`      |
| Behavior Specs       | Input → Output expectations             | "Creating incident sends notification" |
| Acceptance Criteria  | How to verify completion                | "[ ] Employee can create incident"     |
| Constraints & Rules  | Business logic boundaries               | "DL-01: Hours adjustment model"        |
| Authorization Matrix | Who can do what (tables)                | "Employee: create own, read own"       |
| Related Documents    | Links to PRDs, master TDDs, siblings    | "Parent PRD: [link], Backend: [link]"  |
| Open Questions       | Unresolved decisions needing input      | "OQ-01: Retention policy?" - Open      |

## Exclude from TDDs

| Element              | Why Exclude                     | Where It Belongs |
| -------------------- | ------------------------------- | ---------------- |
| Full module code     | Gets stale, duplicates codebase | Implementation   |
| Internal algorithms  | Implementation detail           | Code comments    |
| Function bodies      | Will change during dev          | Codebase         |
| Boilerplate          | Noise, no specification value   | Templates        |
| Test implementations | Duplicates test files           | Test suite       |

## MCP Integration

- **Sequential MCP**: Structured analysis of requirements and systematic TDD construction
- **Context7 MCP**: Framework patterns for data models, authorization patterns
- **Serena MCP**: Project memory for cross-TDD consistency and domain understanding

## Open Questions Generation Workflow

TDD creation follows a **two-step process** where Open Questions are generated separately using deep analysis:

### Step 1: Create TDD Core

Generate all TDD sections EXCEPT Open Questions:

1. Overview and Scope
2. Data Model (tables)
3. Interface Contract (signatures only)
4. Authorization Matrix
5. Behavior Specifications (Given/When/Then)
6. Acceptance Criteria (checkboxes)
7. Related Documents

Write the TDD to file with an empty Open Questions section:

```markdown
## Open Questions

*Generating via deep analysis...*
```

### Step 2: Deep Analysis for Open Questions (ultrathink)

After the TDD core is written, invoke Sequential MCP with `--ultrathink` depth to analyze the complete TDD and generate meaningful Open Questions.

**Analysis Focus Areas**:

| Area                     | Question Types                                        |
| ------------------------ | ----------------------------------------------------- |
| Requirements Ambiguity   | Unclear acceptance criteria, missing edge cases       |
| Stakeholder Input Needed | Business decisions, policy clarifications             |
| Technical Decisions      | Architecture choices, integration approaches          |
| Scope Boundaries         | What's explicitly out of scope, future considerations |
| Data Constraints         | Validation rules, retention policies, limits          |
| Authorization Edge Cases | Role inheritance, delegation, audit requirements      |

**Sequential MCP Prompt Template**:

```
Analyze this TDD for unresolved decisions and ambiguities that require
stakeholder input before implementation can begin.

For each question:
1. Assign a unique ID (OQ-01, OQ-02, etc.)
2. State the question clearly
3. Explain why this needs resolution
4. Suggest possible answers if applicable
5. Mark status as "Open" or "Deferred to v2"

Focus on questions that would BLOCK implementation if left unresolved.
```

**Output Format**:

```markdown
## Open Questions

| ID    | Question                                | Status            |
| ----- | --------------------------------------- | ----------------- |
| OQ-01 | What is the retention policy for logs?  | Open              |
| OQ-02 | Should admins see all user data?        | Deferred to v2    |

### OQ-01: Retention Policy

**Question**: How long should audit logs be retained?

**Why it matters**: Affects storage requirements and compliance obligations.

**Possible answers**:

- [ ] 30 days (minimal compliance)
- [ ] 90 days (standard practice)
- [ ] 1 year (regulatory requirement)

**Status**: Open - needs legal/compliance input
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

- **Backend Resource TDD**: Data models, actions, policies for Ash resources
- **UI TDD**: Routes, screens, components, user flows (no LiveView code)
- **API TDD**: Endpoints, payloads, authentication, error handling
- **Integration TDD**: Cross-domain interactions, event flows, external systems
- **TDD Review Report**: Analysis of existing TDD for code bloat with lean suggestions
- **Consolidated TDD**: Tightened document with OQ answers applied and resolved questions removed

## Examples

### Backend Resource TDD

```
/tdd incidents --type backend --prd @specs/prds/02-incident-management.md

# Generates backend TDD with data model, actions, authorization (no code)
```

### UI TDD

```
/tdd incidents-ui --type ui --prd @specs/prds/02-incident-management.md

# Generates UI TDD with routes, screens, components (no LiveView code)
```

### API TDD

```
/tdd payment-webhook --type api --iterative

# Interactive session to clarify endpoints, payloads, error handling
```

### Review Existing TDD

```
/tdd --review @specs/tdds/incidents/01-incident-resource.md

# Analyzes TDD for code bloat, suggests lean improvements
```

### Consolidate After OQ Discussion

```
/tdd --consolidate @specs/tdds/incidents/01-incident-resource.md

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
3. **Update Open Questions**:
   - Mark resolved OQs with status "Resolved" and brief answer
   - Remove fully integrated OQs from the table
   - Keep unresolved OQs visible
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

## Open Questions

*All questions resolved and integrated.*
```

### When to Consolidate

- After answering 3+ Open Questions in discussion
- Before starting implementation
- When TDD context feels bloated
- Before sharing TDD with stakeholders

## Complexity Management

### When to Split TDDs

If a TDD exceeds these thresholds, suggest splitting into multiple documents:

| Indicator                | Threshold              | Action                     |
| ------------------------ | ---------------------- | -------------------------- |
| Data models              | > 3 distinct resources | Split by domain            |
| Acceptance criteria      | > 25 checkboxes        | Split by feature area      |
| Behavior specs           | > 10 scenarios         | Split by user journey      |
| Estimated implementation | > 1 week               | Split by deliverable       |
| Document length          | > 500 lines            | Review for bloat           |
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
- [ ] No "implementation details" or "how to implement" sections
- [ ] Authorization uses matrix format
- [ ] Behavior specs use Given/When/Then
- [ ] Open Questions generated via Sequential MCP + ultrathink (Phase 2 complete)
- [ ] Each OQ has ID, rationale, possible answers, and status
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
