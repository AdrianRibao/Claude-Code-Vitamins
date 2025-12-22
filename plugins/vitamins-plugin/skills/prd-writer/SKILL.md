---
name: prd-writer
description: Generate excellent Product Requirements Documents focused on problems, goals, users, and success metrics. Use when creating PRDs, defining product scope, or documenting feature requirements.
allowed-tools: Read, Grep, Glob, Write, Edit, AskUserQuestion, TodoWrite, Task
---

# PRD Writer Skill

> **Philosophy**: PRDs define WHAT problem we're solving and WHY, not HOW to implement it. Technical details belong in TDDs.

## When to Use

- New product or feature requiring clear requirements definition
- Stakeholder alignment on scope, goals, and success criteria
- Complex features needing structured requirements discovery
- Before creating TDDs - PRDs are the parent specification
- PRD review requests to identify scope creep or missing requirements
- Consolidating PRD after Open Questions are answered

## Usage

```
/prd [feature-name] [--type master|feature|api|integration] [--no-questions] [--review] [--consolidate @path]
```

| Flag | Purpose |
|------|---------||
| `--type` | PRD type: master, feature, api, integration |
| `--no-questions` | Skip upfront questions (use with comprehensive brief) |
| `--review` | Analyze existing PRD for scope creep or gaps |
| `--consolidate @path` | Apply OQ answers, tighten document |

## Output Location

PRDs are created at:

```
specs/prds/{product}/{nn}-{product}-{type}.md
```

**Examples:**

| Command | Output Path |
|---------|-------------|
| `/prd user-auth --type master` | `specs/prds/user-auth/00-user-auth-master.md` |
| `/prd user-auth-login --type feature` | `specs/prds/user-auth/01-user-auth-login.md` |
| `/prd user-auth-api --type api` | `specs/prds/user-auth/02-user-auth-api.md` |

**Numbering convention:**
- `00-` Master PRD (product overview, document hierarchy)
- `01+` Feature PRDs (specific features, ordered by priority)

Ask user to confirm or override path during Phase 0 questions.

## Resources

- [style-guide.md](style-guide.md) - PRD writing conventions and rules
- [templates/](templates/) - PRD templates by type (master, feature)
- [examples/](examples/) - Reference PRD examples

## Behavioral Mindset

Requirements-first approach: every section answers "what" and "why", never "how". Use tables for scannable requirements. Write clear user workflows with step-by-step scenarios. Create measurable success metrics. Never include code or implementation details.

**Always ask clarifying questions before generating a PRD.** Understanding the problem deeply prevents scope creep and produces better specifications.

## Key Actions

### Phase 0: Requirements Discovery (Always)

1. **Read Context**: Review existing documents, related PRDs, market context
2. **Identify Problem**: Understand the core problem being solved
3. **Ask Questions**: Use `AskUserQuestion` to clarify before writing
   - What problem are we solving? For whom?
   - What are the primary goals and non-goals?
   - Who are the target users and their key workflows?
   - What does success look like? How will we measure it?
   - What constraints exist (timeline, resources, technical)?
   - What's explicitly out of scope?
4. **Confirm Understanding**: Summarize answers before proceeding

**Skip questioning only if:** User provides comprehensive brief AND explicitly says "no questions needed".

### Phase 1: PRD Core Generation

5. **Discover**: Analyze problem space, understand user needs, identify scope boundaries
6. **Map Document Hierarchy**: Identify parent PRDs, child feature PRDs, related TDDs
7. **Analyze Existing Patterns**: Find related PRDs in the codebase for consistency
8. **Structure**: Organize into logical sections following lean PRD template
9. **Specify**: Write problem statement, goals, user workflows, requirements, success metrics
10. **Link Documents**: Add "Related Documents" section with all relevant PRDs and TDDs
11. **Write PRD**: Save to file with placeholder Open Questions section

### Phase 2: Open Questions Deep Analysis

12. **Invoke Sequential MCP**: Use `--ultrathink` for maximum depth analysis
13. **Analyze Gaps**: Identify ambiguities, missing decisions, edge cases, scope boundaries
14. **Generate Questions**: Create structured OQ entries with IDs, rationale, and possible answers
15. **Append to PRD**: Update the Open Questions section with generated content

### Phase 3: Finalization

16. **Update References**: Update master PRDs and related documents to reference the new PRD
17. **Validate**: Review for clarity, completeness, and measurability
18. **Quality Check**: Ensure no implementation details, all requirements testable, all links valid
19. **Present for Review**: Show complete PRD to user, await approval before TDD creation

## Include in PRDs

| Element | Purpose | Example |
|---------|---------|---------|
| Problem Statement | WHY this exists | "Users struggle to track their hours worked" |
| Goals | SUCCESS looks like | "Reduce time-tracking errors by 50%" |
| Non-Goals | What we're NOT doing | "Not replacing payroll system" |
| Target Users | WHO we're building for | "Household employees and employers" |
| User Workflows | HOW users accomplish tasks | Step-by-step scenarios with time estimates |
| Requirements | Functional needs | "System must calculate overtime automatically" |
| Success Metrics | HOW we measure | "90% user adoption within 3 months" |
| Scope Boundaries | IN/OUT of scope | "In: Time tracking. Out: Payroll processing" |
| Constraints | LIMITATIONS we accept | "Must work offline on mobile" |
| Open Questions | Unresolved decisions | "OQ-01: Should we support multiple employers?" |

## Exclude from PRDs

| Element | Why Exclude | Where It Belongs |
|---------|-------------|------------------|
| Implementation code | PRDs define WHAT, not HOW | TDD, Codebase |
| Database schemas | Technical detail | TDD |
| API specifications | Technical detail | TDD (API type) |
| UI wireframes | Unless high-level concepts | Design docs |
| Architecture decisions | Technical detail | TDD, ADRs |
| Deployment plans | Operations detail | DevOps docs |

## MCP Integration

- **Sequential MCP**: Structured analysis of requirements and systematic PRD construction
- **Context7 MCP**: Industry patterns, competitive analysis, best practices
- **Serena MCP**: Project memory for cross-PRD consistency and domain understanding

## Open Questions Generation Workflow

PRD creation follows a **two-step process** where Open Questions are generated separately using deep analysis:

### Step 1: Create PRD Core

Generate all PRD sections EXCEPT Open Questions:

1. Problem Statement & Background
2. Goals and Non-Goals
3. Target Users & Personas
4. User Workflows (step-by-step scenarios)
5. Requirements (Functional, Non-Functional)
6. Success Metrics & KPIs
7. Scope (In/Out)
8. Constraints & Dependencies
9. Related Documents

Write the PRD to file with an empty Open Questions section:

```markdown
## Open Questions

*Generating via deep analysis...*
```

### Step 2: Deep Analysis for Open Questions (ultrathink)

After the PRD core is written, invoke Sequential MCP with `--ultrathink` depth to analyze the complete PRD and generate meaningful Open Questions.

**Analysis Focus Areas**:

| Area | Question Types |
|------|----------------|
| Problem Clarity | Is the problem well-defined? Are we solving the right problem? |
| User Understanding | Do we understand all user segments? Edge cases? |
| Scope Boundaries | What should explicitly be out of scope? Future phases? |
| Success Measurement | Are metrics measurable? Do we have baselines? |
| Stakeholder Input | Business decisions, policy clarifications needed? |
| Dependencies | External dependencies? Integration requirements? |
| Risk Factors | What could prevent success? Mitigation strategies? |

**Sequential MCP Prompt Template**:

```
Analyze this PRD for unresolved decisions and ambiguities that require
stakeholder input before TDD creation can begin.

For each question:
1. Assign a unique ID (OQ-01, OQ-02, etc.)
2. State the question clearly
3. Explain why this needs resolution
4. Suggest possible answers if applicable
5. Mark status as "Open" or "Deferred to v2"

Focus on questions that would BLOCK TDD creation if left unresolved.
```

**Output Format**:

```markdown
## Open Questions

| ID | Question | Status |
|----|----------|--------|
| OQ-01 | Should we support multiple employers per employee? | Open |
| OQ-02 | What's the retention period for time records? | Deferred to v2 |

### OQ-01: Multiple Employers Support

**Question**: Should an employee be able to work for multiple employers simultaneously?

**Why it matters**: Affects data model, UI complexity, and reporting requirements.

**Possible answers**:

- [ ] Single employer only (simpler, v1 scope)
- [ ] Multiple employers (more complex, broader market)
- [ ] Multiple employers with primary designation

**Status**: Open - needs product decision
```

### Automatic Execution

Both steps run automatically with a single `/prd` command:

```
/prd [feature-name] [--type master|feature|api|integration]

Execution:
  Step 1: Generate PRD core sections → Write to file
  Step 2: Invoke Sequential MCP + ultrathink → Analyze PRD → Append Open Questions
  Result: Complete PRD with deep-analysis-generated questions
```

## Outputs

- **Master PRD**: Product overview, vision, document hierarchy, cross-cutting requirements
- **Feature PRD**: Specific feature requirements, user workflows, success metrics
- **API PRD**: API product requirements, consumer needs, integration patterns
- **Integration PRD**: External system integration requirements, data flows
- **PRD Review Report**: Analysis of existing PRD for scope creep with lean suggestions
- **Consolidated PRD**: Tightened document with OQ answers applied and resolved questions removed

## Examples

### Master PRD

```
/prd time-tracking --type master

# Generates master PRD with product vision, scope, document hierarchy
```

### Feature PRD

```
/prd time-tracking-mobile --type feature

# Generates feature PRD with user workflows, requirements, success metrics
```

### Review Existing PRD

```
/prd --review @specs/prds/time-tracking/00-master.md

# Analyzes PRD for scope creep, missing requirements, suggests improvements
```

### Consolidate After OQ Discussion

```
/prd --consolidate @specs/prds/time-tracking/01-mobile.md

# Applies OQ answers from conversation, tightens document, verifies thresholds
```

## Post-Creation Workflow

After creating a PRD:

1. **Present for review**: Show the complete PRD to the user
2. **Await approval**: Do NOT start TDD creation until user approves
3. **Resolve Open Questions**: All OQ items should be addressed before TDD creation
4. **Update if needed**: Incorporate user feedback into the PRD

**CRITICAL**: Creating a PRD is a specification phase, NOT a TDD trigger. Always read and present the PRD for review before any TDD creation begins.

## Consolidation Workflow

After Open Questions are answered in discussion, use `--consolidate` to apply answers and tighten the document.

### Usage

```
/prd --consolidate @specs/prds/product-feature.md
```

### Consolidation Actions

1. **Read Context**: Parse PRD and recent conversation for OQ answers
2. **Apply Answers**: Integrate resolved decisions into relevant sections
   - Update Goals with clarified success criteria
   - Add missing User Workflows for resolved edge cases
   - Adjust Requirements based on scope decisions
   - Update Success Metrics with agreed baselines
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

| ID | Question | Status |
|----|----------|--------|
| OQ-01 | Support multiple employers? | Open |
| OQ-02 | Maximum hours per week? | Open |
```

**After consolidation** (user answered: single employer v1, 60 hours max):
```markdown
## Scope

| In Scope | Out of Scope |
|----------|--------------|
| Single employer per employee | Multiple employer support (v2) |  <!-- Applied from OQ-01 -->

## Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-05 | System enforces 60-hour weekly maximum | P1 |  <!-- Applied from OQ-02 -->

## Open Questions

*All questions resolved and integrated.*
```

### When to Consolidate

- After answering 3+ Open Questions in discussion
- Before creating TDDs from the PRD
- When PRD context feels bloated
- Before sharing PRD with stakeholders

## Complexity Management

### When to Split PRDs

If a PRD exceeds these thresholds, suggest splitting into multiple documents:

| Indicator | Threshold | Action |
|-----------|-----------|--------|
| User workflows | > 8 workflows | Split by user segment or journey |
| Requirements | > 30 requirements | Split by feature area |
| Target users | > 4 distinct personas | Split by persona |
| Document length | > 400 lines | Review for bloat |
| Document length | > 1000 lines | **Hard limit** - must split |

### Master PRD Pattern

For complex products, create a document hierarchy:

1. **Master PRD** (`00-product-master.md`): Vision, scope, document hierarchy, cross-cutting requirements
2. **Child PRDs** by feature:
    - `01-product-feature-a.md` - First feature PRD
    - `02-product-feature-b.md` - Second feature PRD
    - `03-product-api.md` - API requirements (if applicable)
    - `04-product-integration.md` - Integration requirements (if applicable)

Each child PRD links back to the master and to sibling PRDs.

## Quality Checklist

Before finalizing a PRD, verify:

- [ ] Problem statement is clear and compelling
- [ ] Goals are specific, measurable, achievable
- [ ] Non-goals explicitly define what we're NOT doing
- [ ] Target users are well-defined with clear needs
- [ ] User workflows are step-by-step scenarios with time estimates
- [ ] Requirements are testable and prioritized
- [ ] Success metrics have baselines and targets
- [ ] Scope boundaries are crystal clear
- [ ] No implementation details or code
- [ ] No technical architecture decisions
- [ ] Open Questions generated via Sequential MCP + ultrathink (Phase 2 complete)
- [ ] Each OQ has ID, rationale, possible answers, and status
- [ ] PRD presented for user review before TDD creation
- [ ] Large PRDs split with master document pattern

## Boundaries

**Will:**

- **Always ask clarifying questions before generating** (unless `--no-questions`)
- Generate lean PRDs focused on problems, goals, and requirements
- Use tables and structured formats over prose
- Create measurable success criteria
- Reference existing patterns in the codebase
- Identify and suggest removal of scope creep in reviews
- Consolidate PRDs by applying OQ answers and tightening documents

**Will Not:**

- Include implementation code or technical details
- Write database schemas or API specifications
- Make architecture decisions
- Create UI wireframes or detailed designs
- Generate TDDs (separate skill/phase)
