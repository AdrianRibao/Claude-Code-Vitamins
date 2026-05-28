# TDD — Marketing Board: Deliberation

| Field       | Value                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------- |
| Type        | Feature TDD                                                                                                          |
| Status      | v1.1 — OQ-generation contract tightened to mirror PRD/TDD skill Phase 2                                              |
| Owner       | Adrián Ribao                                                                                                         |
| Created     | 2026-05-20                                                                                                           |
| Parent TDD  | [`00-marketing-board-master.md`](00-marketing-board-master.md)                                                       |
| Parent PRD  | [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md) |
| Sibling TDD | [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md)                                                 |

______________________________________________________________________

## Overview

Specifies the contracts for the **deliberation flow**: the `/marketing-board` slash command, the 8 subagents it dispatches, the synthesizer prompt that combines their reports, and the Marketing Plan Memo it produces. Cross-cutting concerns (file layout, naming, knowledge-base shape) live in the master TDD. Bootstrap (knowledge-base creation) lives in TDD 02.

## Scope

**In:**

- `commands/marketing-board.md` — `/marketing-board <brief>` command
- 8 agent files under `agents/`
- Synthesizer logic embedded in the command body
- Marketing Plan Memo output format
- `--consolidate` flag contract and automatic memo persistence

**Out:**

- Bootstrap skill, INTERVIEW.md, knowledge-base file generation → TDD 02
- Knowledge-base file shapes (data contracts) → TDD 00
- v2 production skills (email sequences, video scripts, LP audit)

## Slash command contract — `commands/marketing-board.md`

### Frontmatter

| Field           | Value                                                                                                                                                  |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `description`   | "Convene 8 marketing specialists on a SaaS launch brief; synthesize a Marketing Plan Memo. Requires `agent_docs/marketing/` (run `:bootstrap` first)." |
| `argument-hint` | `<launch brief \| --consolidate [@memo]>`                                                                                                              |

### Body — required step sequence

The command body MUST execute these steps **in order**:

| Step | Action                             | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | **Verify knowledge base**          | Check `agent_docs/marketing/` for all 6 required files. If any missing → hard-fail (see FR-15).                                                                                                                                                                                                                                                                                                                                                          |
| 2    | **Parse flags**                    | Detect `--consolidate` in `$ARGUMENTS`; route to the consolidate sub-flow if matched.                                                                                                                                                                                                                                                                                                                                                                    |
| 3    | **Frame the decision**             | Restate the brief; pull repo-aware context via Read/Grep/Glob when brief references project files.                                                                                                                                                                                                                                                                                                                                                       |
| 4    | **Dispatch agents in parallel**    | Single Agent-tool fan-out invoking all 8 subagents by name, each with the framed brief.                                                                                                                                                                                                                                                                                                                                                                  |
| 5    | **Synthesize memo + OQs**          | Load synthesizer prompt from `reference/synthesizer-prompt.md` (per OQ-D-05); read all 8 reports; apply attribution convention; produce memo per template (below) **in the same Claude turn** (per OQ-D-01). The memo's final section (`## Open Questions`) is generated via deep analysis following the **Open Questions generation contract** — same shape the prd-writer / tdd-writer skills produce in their Phase 2 (table + per-OQ detail blocks). |
| 6    | *(merged into step 5 per OQ-D-01)* | OQ section is emitted inline as the final memo section in the same turn. No separate Claude call.                                                                                                                                                                                                                                                                                                                                                        |
| 7    | **Save memo**                      | Always write the memo to `marketing-plans/<product>-<YYYY-MM-DD>.md` (create `marketing-plans/` if absent; same-day collision → `-<HHMMSS>` suffix). No flag — persistence is automatic (per FR-20).                                                                                                                                                                                                                                                     |

### Flag contracts

| Flag            | Args           | Sub-flow                                                                                                                                                                                                                                                        |
| --------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *(none)*        | `<brief>`      | Full deliberation flow (steps 1-7 above); the memo is written to disk at step 7                                                                                                                                                                                 |
| `--consolidate` | `[@memo-path]` | Read the memo **file** (path arg, or most recently written file in `marketing-plans/`, per OQ-D-02); apply the OQ answers the user marked in the file; mark Resolved; tighten prose; write the updated memo back to the same file. Skip steps 4-7 of main flow. |

### Hard-fail contract (FR-15)

| Condition                                                                                               | Action                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_docs/marketing/` doesn't exist OR any of 6 required files missing                                | Abort at step 1 with **exact** message: `"Bootstrap your product context first: /marketing-board:bootstrap"` (v1 is English-only — `--lang` arrives in v1.1 per TDD 02 OQ-B-06) |
| Required files exist but a `##` section is missing                                                      | Abort with: `"Knowledge base file <file>.md is missing required section: <section>. Re-run /marketing-board:bootstrap --consolidate."`                                          |
| `--consolidate` invoked but no memo file found (no path arg, and `marketing-plans/` is empty or absent) | Abort with: `"No memo found to consolidate. Run /marketing-board <brief> first."`                                                                                               |

No partial-context fallback. No silent degradation.

## Agent contracts (8 seats)

### Common frontmatter (all 8)

| Field         | Convention / value                                  |
| ------------- | --------------------------------------------------- |
| `name`        | Matches agent slug (see TDD 00 naming conventions)  |
| `description` | One-sentence lens declaration, sharp and orthogonal |
| `tools`       | `Read, Grep, Glob, WebSearch, WebFetch`             |
| `model`       | Per-seat (see table below)                          |

No `permissionMode`, `mcpServers`, `hooks`, `memory`, `skills`, or `background` fields (per Architecture Decision 8 + plugin-subagent restrictions).

### Per-seat configuration

| Slug                 | Model    | Required knowledge-base files                                                             |
| -------------------- | -------- | ----------------------------------------------------------------------------------------- |
| `ethnographer`       | sonnet   | `product.md`, `audience.md`                                                               |
| `storyteller`        | **opus** | `product.md`, `audience.md`, `competition.md`                                             |
| `channel-strategist` | sonnet   | `product.md`, `audience.md`, `business.md`, `distribution.md`                             |
| `producer`           | sonnet   | `product.md`, `audience.md`, `distribution.md`                                            |
| `funnel-engineer`    | sonnet   | `product.md`, `business.md`                                                               |
| `community-pr-lead`  | sonnet   | `product.md`, `audience.md`, `distribution.md`                                            |
| `moonshot`           | **opus** | `product.md`, `competition.md`, `constraints.md`                                          |
| `contrarian`         | **opus** | All six (`product`, `audience`, `competition`, `business`, `distribution`, `constraints`) |

### Common system-prompt structure (markdown body)

Each agent's Markdown body MUST contain these sections in this order:

| Section              | Purpose                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `## Identity`        | "You are the [Seat] of the Marketing Board." 1-2 sentences.                                      |
| `## Your lens`       | Bullet list, 4-7 angles the agent looks for                                                      |
| `## How you think`   | Bullet list, 3-5 reasoning principles (e.g. "Steel-man the opposition")                          |
| `## Context loading` | Explicit instruction: "At the start, read `agent_docs/marketing/<files>` from the project root." |
| `## Output format`   | Markdown skeleton the synthesizer can parse (per-seat; see below)                                |

No other top-level sections in v1.

### Per-seat output-format skeleton

Each seat's report has a fixed shape so the synthesizer can extract a one-line position for the verdicts table and locate disagreement points.

**H2 wrapper convention (OQ-D-03)**: every agent's report MUST open with `## <Seat> Report` (H2) as the **first line**, followed by the per-seat `###` sub-headers below. This wrapper makes reports self-contained chunks the synthesizer can extract by H2 boundary.

| Seat               | Required `###` sub-headers (under top-level `## <Seat> Report`)                                                                                                                                       |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ethnographer       | `### One-line position` · `### ICP refinement` · `### Personas` · `### Attention map` · `### Vocabulary insights`                                                                                     |
| Storyteller        | `### One-line position` · `### Positioning` · `### Message house` · `### Hero narrative` · `### Taglines`                                                                                             |
| Channel Strategist | `### One-line position` · `### Channel mix` · `### Budget allocation` · `### Sequencing` · `### Expected CAC`                                                                                         |
| Producer           | `### One-line position` · `### Editorial calendar` · `### Asset list` · `### Production briefs` · `### Distribution map`                                                                              |
| Funnel Engineer    | `### One-line position` · `### Funnel map` · `### Email sequences (outlines)` · `### A/B priorities` · `### Retention loops`                                                                          |
| Community/PR Lead  | `### One-line position` · `### Communities map` · `### Ambassadors / creators list` · `### Launch-day sequence` · `### PR pitch angles`                                                               |
| Moonshot           | `### Reframing` · `### Asymmetric bets (1-3)` · `### Expected payoff` · `### Test cost`                                                                                                               |
| Contrarian         | `### Verdict` · `### Strongest case against` · `### Assumptions that kill the thesis` · `### Base rate / outside view` · `### Failure mode that looks like success` · `### What would change my mind` |

The synthesizer relies on `### One-line position` (or `### Verdict` for Contrarian, `### Reframing` for Moonshot) being the **first sub-header** in every report — that's what populates the verdicts-at-a-glance table.

## Synthesizer prompt structure

The synthesizer prompt lives in **`plugins/marketing-board/reference/synthesizer-prompt.md`** (per OQ-D-05). The slash command body loads it via `Read` at step 5 and includes its content as additional instructions for the main Claude turn that produces the memo. This keeps the prompt editable and versioned independently of `commands/marketing-board.md` (per NFR-06).

The synthesizer is **not a separate agent** (per Architecture Decision 1 — subagents can't spawn subagents). It's instructions executed in the main session after agents return. The reference-file location is a code-organization choice, not an architectural one.

After loading the reference prompt at step 5, the main session executes:

| Synthesizer step | Action                                                                                                                                                                                                                                                                                         |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| S-1              | Read all 8 reports from the Agent-tool return values                                                                                                                                                                                                                                           |
| S-2              | Extract each seat's one-line position (from `### One-line position` / `### Verdict` / `### Reframing` header)                                                                                                                                                                                  |
| S-3              | Populate the **Board verdicts at a glance** table with the 8 one-liners                                                                                                                                                                                                                        |
| S-4              | Identify convergent points across reports → populate **Where the board agrees**                                                                                                                                                                                                                |
| S-5              | Identify conflicting positions → populate **Where the board disagrees** (attributed per OQ-10: name the seats taking each side)                                                                                                                                                                |
| S-6              | Synthesize a single-voice **plan I'm proposing**, addressing the strongest Contrarian objection explicitly                                                                                                                                                                                     |
| S-7              | Lift Contrarian's strongest objection into **What I'm accepting as risk**                                                                                                                                                                                                                      |
| S-8              | Build **Pre-launch checklist** from the cross-seat must-haves                                                                                                                                                                                                                                  |
| S-9              | Build **First 30 days** from Channel Strategist + Producer + Community/PR Lead near-term recommendations                                                                                                                                                                                       |
| S-10             | Build **Metrics to track** from Funnel Engineer + Channel Strategist KPIs                                                                                                                                                                                                                      |
| S-11             | Generate Open Questions section inline per the **Open Questions generation contract** below (mirrors prd-writer / tdd-writer skill Phase 2) — same Claude turn as memo synthesis (per OQ-D-01), deep-analysis pass over memo + raw reports, locked render shape (table + per-OQ detail blocks) |

### Attribution convention (Architecture Decision 10)

- **Single voice** in: `Brief recap`, `The plan I'm proposing`, `What I'm accepting as risk`, `Pre-launch checklist`, `First 30 days`, `Metrics to track`
- **Attributed** (seat names called out) in: `Where the board disagrees` and (informally, when useful for traceability) `Where the board agrees`

## Marketing Plan Memo template (locked output shape)

Same as PRD §"Marketing Plan Memo — Output Structure (v1)". Restated for testability:

| Section header                        | Body shape                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `# Marketing Plan: [Product]`         | Title with product name                                                                                                                                                                                                                                                                                                                                                                                    |
| `## Brief recap`                      | 1 paragraph                                                                                                                                                                                                                                                                                                                                                                                                |
| `## Board verdicts at a glance`       | 8-row table: Seat \| One-line position. A failed seat (per OQ-D-04 retry-then-flag) shows `[FAILED: <reason>]` instead of its one-liner                                                                                                                                                                                                                                                                    |
| `## Where the board agrees`           | Bullet list                                                                                                                                                                                                                                                                                                                                                                                                |
| `## Where the board disagrees`        | Bullet list with seat attributions                                                                                                                                                                                                                                                                                                                                                                         |
| `## The plan I'm proposing`           | Prose + actionable bullets; single voice                                                                                                                                                                                                                                                                                                                                                                   |
| `## What I'm accepting as risk`       | 1-3 sentences                                                                                                                                                                                                                                                                                                                                                                                              |
| `## Pre-launch checklist`             | Bullet list                                                                                                                                                                                                                                                                                                                                                                                                |
| `## First 30 days — concrete actions` | Numbered list, 3-7 items                                                                                                                                                                                                                                                                                                                                                                                   |
| `## Metrics to track`                 | Table: KPI \| baseline (if known) \| target window                                                                                                                                                                                                                                                                                                                                                         |
| `## Open Questions`                   | Generated by deep-analysis pass (synthesizer step S-11). **Mirrors prd-writer / tdd-writer skill Phase 2 verbatim** — see "Open Questions generation contract" below. Same render shape as this TDD's own OQ section: intro line → summary table (`ID \| Question \| Status`) → per-OQ detail block (`**Question:**`, `**Why it matters:**`, `**Possible answers:**` as `- [ ]` checkboxes, `**Status:**`) |

### Open Questions generation contract (mirrors PRD/TDD skill Phase 2)

The memo's `## Open Questions` section is generated by the same deep-analysis pattern the prd-writer and tdd-writer skills use in their own Phase 2. The synthesizer prompt (in `reference/synthesizer-prompt.md`) MUST instruct Claude to:

| Aspect                  | Spec                                                                                                                                                                                                                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Trigger**             | Same Claude turn as memo synthesis (per OQ-D-01) — no separate sub-prompt                                                                                                                                                                                                                                     |
| **Inputs**              | The synthesized memo + all 8 raw agent reports                                                                                                                                                                                                                                                                |
| **Reasoning depth**     | Ultrathink / deep analysis — the prompt explicitly asks for it                                                                                                                                                                                                                                                |
| **Focus areas**         | Ambiguities, missing decisions, edge cases, scope boundaries, stakeholder-input needs. **Only questions that would block execution if unresolved**                                                                                                                                                            |
| **Skip**                | Cosmetic questions, hypothetical edge cases the brief doesn't motivate, questions already answered in the brief or knowledge base                                                                                                                                                                             |
| **Numbering**           | `OQ-01`, `OQ-02`, ... per-memo, not global                                                                                                                                                                                                                                                                    |
| **Output structure**    | (1) one-line intro `*Generated via deep analysis after synthesis — resolve before execution. Answer these in this file (tick the checkboxes), then run /marketing-board --consolidate to fold them in.*`; (2) summary table with columns `ID \| Question \| Status`; (3) one per-OQ detail block for each row |
| **Per-OQ detail block** | Exactly this shape, mirroring the skill format used throughout this spec family:                                                                                                                                                                                                                              |

```markdown
### OQ-NN — <Question title>

**Question:** <one-sentence statement>

**Why it matters:** <one line — what unblocks once resolved>

**Possible answers:**

- [ ] Option A — short rationale
- [ ] Option B — short rationale
- [ ] Option C — short rationale

**Status:** Open  *(or `Deferred to v2` when applicable)*
```

The synthesizer prompt may include its own recommendation at the end of each `**Status:**` line (e.g. `Open — recommend Option B because…`), mirroring how we've handled OQ recommendations throughout the PRD and TDDs in this family.

**Why this matters as a locked contract**: the `/marketing-board --consolidate` flow (and the user's manual OQ-answering pattern) both depend on this exact shape. Drift breaks the consolidation parser and breaks user habit built up across PRD/TDD work.

## `--consolidate` flag contract

Mirrors `/prd --consolidate` from the prd-writer skill. The user answers OQ items **in the saved memo file** — ticking the `- [ ]` checkbox on the chosen option, optionally adding a note. `--consolidate` then folds those answers in:

| Consolidator step | Action                                                                                                                                                |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| C-1               | Locate the memo file (path arg `@<file>`, or the most recently written file in `marketing-plans/`)                                                    |
| C-2               | Read the file; parse the OQ section — a question is answered when one of its `- [ ]` options is ticked `- [x]`, and/or the user added a note under it |
| C-3               | For each answered OQ, integrate the answer into the relevant memo section                                                                             |
| C-4               | Mark the OQ row `Resolved (YYYY-MM-DD)` and add a 1-line resolution summary in the detail block                                                       |
| C-5               | Optionally collapse the OQ section to a resolution log if all OQs resolved                                                                            |
| C-6               | Write the consolidated memo **back to the same file** (in place)                                                                                      |

Consolidate is **idempotent**: running it twice on the same memo file + same in-file answers produces a byte-identical file.

## Memo persistence (automatic — step 7)

Every deliberation writes its memo to disk at step 7. There is no flag — persistence is always on (per FR-20 / PRD Architecture Decision 9).

| Aspect                  | Spec                                                                                                                                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Target path             | `marketing-plans/<product-slug>-<YYYY-MM-DD>.md` in the current working directory                                                                                                                                  |
| Product-slug derivation | The product name — proper-noun subject of `product.md`'s `## Product description` — kebab-cased (e.g. `empleo.digital` → `empleo-digital`). Falls back to the working-directory basename if no clear name is found |
| Directory creation      | Create `marketing-plans/` if absent                                                                                                                                                                                |
| Filename collision      | Append `-<HHMMSS>` if a same-date file already exists — a re-run never overwrites an annotated memo                                                                                                                |
| Trigger                 | Automatic on every deliberation. `--consolidate` writes back to the existing file in place rather than creating a new one                                                                                          |

## Behavior specifications

### Happy path

**Given** `agent_docs/marketing/` exists with all 6 required files, all sections present
**When** user runs `/marketing-board <brief>`
**Then** 8 agents dispatch in parallel within one Agent-tool fan-out
**And** each agent loads its required files and produces a structured report
**And** the synthesizer produces a Marketing Plan Memo following the template
**And** the OQ section is appended after the memo
**And** the memo is written to `marketing-plans/<product-slug>-<YYYY-MM-DD>.md` at step 7

### Missing knowledge base

**Given** `agent_docs/marketing/audience.md` is missing
**When** user runs `/marketing-board <brief>`
**Then** the command aborts at step 1
**And** the exact hard-fail message (FR-15) is displayed
**And** no agents are dispatched

### Knowledge-base section missing

**Given** `agent_docs/marketing/audience.md` exists but `## Personas` header is absent
**When** user runs `/marketing-board <brief>`
**Then** the command aborts at step 1 with the section-missing message
**And** the message names the file and missing section

### Consolidation

**Given** a saved memo file exists in `marketing-plans/` with 4 Open Questions
**And** the user has ticked a `- [x]` answer on all 4 OQs in that file
**When** user runs `/marketing-board --consolidate`
**Then** the consolidator locates the most recently written memo file
**And** updates the memo's relevant sections with the 4 resolutions
**And** marks each OQ `Resolved (2026-05-20)`
**And** writes the consolidated memo back to the same file

### Memo persistence (automatic)

**Given** a deliberation runs to completion for the empleo.digital project
**When** the synthesizer finishes the memo at step 7
**Then** `marketing-plans/empleo-digital-2026-05-20.md` is created in the working directory
**And** the file content matches the memo exactly
**And** a second deliberation the same day is written to `marketing-plans/empleo-digital-2026-05-20-<HHMMSS>.md`, leaving the first file untouched

### Single-seat invocation (Workflow 3)

**Given** all knowledge-base files exist
**When** user types `@agent-marketing-board:ethnographer <brief>`
**Then** only the Ethnographer is dispatched
**And** it produces its standard report
**And** no synthesizer step runs

### Agent failure during deliberation (OQ-D-04)

**Given** all knowledge-base files exist
**And** one agent (e.g. Moonshot) returns an error or times out during the parallel fan-out
**When** the dispatch step detects the failure
**Then** the command **retries that agent once**
**And** if retry succeeds → proceeds with all 8 reports (happy path)
**And** if retry fails → proceeds with **N-1 reports** and the synthesizer emits the failed seat's verdicts-table row as `[FAILED: <retry exhausted: <reason>>]`
**And** the synthesizer notes the failure once in the memo's `Brief recap` so the user knows the deliberation was partial
**And** the OQ section includes a `[verify retry]` callout flagging the failure

## Acceptance criteria

### Plugin structure

- [x] `plugins/marketing-board/.claude-plugin/plugin.json` exists with required fields (TDD 00)
- [x] `plugins/marketing-board/commands/marketing-board.md` exists with frontmatter per spec
- [x] All 8 agent files exist at `plugins/marketing-board/agents/<slug>.md`
- [x] Plugin is registered in `.claude-plugin/marketplace.json`

### Agent contracts

- [x] Each agent's frontmatter has `name`, `description`, `tools`, `model`
- [x] `tools` is exactly `Read, Grep, Glob, WebSearch, WebFetch` for all 8
- [x] `model: opus` for Storyteller, Moonshot, Contrarian; `model: sonnet` for the other five
- [x] Each agent body contains the 5 required sections (`## Identity`, `## Your lens`, `## How you think`, `## Context loading`, `## Output format`)
- [x] `## Context loading` names the specific `agent_docs/marketing/*.md` files the agent reads
- [x] `## Output format` matches the per-seat skeleton in this TDD

### Command behavior

- [ ] Hard-fail triggers correctly when `agent_docs/marketing/` is missing
- [ ] Hard-fail triggers correctly when any required file is missing
- [ ] Hard-fail triggers correctly when a required `##` section is missing
- [ ] Happy-path deliberation dispatches all 8 agents in parallel (single Agent-tool fan-out, not 8 sequential calls)
- [x] Memo follows the locked template structure (all required sections present)
- [x] Verdicts-at-a-glance table has exactly 8 rows (or N-1 with `[FAILED:]` row on agent failure)
- [x] Disagreements section attributes positions to seats
- [x] OQ section is emitted inline as the final memo section in the same Claude turn (per OQ-D-01)
- [ ] OQ section structure matches the **Open Questions generation contract** verbatim — intro line, summary table (`ID | Question | Status`), one per-OQ detail block per row with `**Question:** / **Why it matters:** / **Possible answers:** (`- [ ]` list) / **Status:**` (mirrors PRD/TDD skill Phase 2)
- [x] Each OQ targets a question that would block execution if unresolved (not cosmetic, not already answered in brief/KB)
- [ ] Each agent's report opens with `## <Seat> Report` H2 wrapper (per OQ-D-03)

### Failure handling

- [ ] Single-agent failure triggers one retry; retry success continues normally
- [ ] Double-failure (initial + retry) proceeds with N-1; failed seat shows `[FAILED:]` in verdicts table
- [ ] Brief recap of partial-deliberation memos notes the failure
- [ ] OQ section includes `[verify retry]` callout on partial deliberations

### Synthesizer prompt file

- [x] `plugins/marketing-board/reference/synthesizer-prompt.md` exists
- [x] `commands/marketing-board.md` loads it via `Read` at step 5

### Memo persistence & `--consolidate`

- [x] Every deliberation writes the memo to `marketing-plans/<slug>-<YYYY-MM-DD>.md` at step 7
- [x] `marketing-plans/` directory is created if absent
- [ ] A same-day second deliberation gets a `-<HHMMSS>` suffix; the earlier file is not overwritten
- [x] `--consolidate` reads the OQ answers the user marked in the saved memo file (ticked `- [x]` checkboxes / inline notes)
- [x] `--consolidate` resolves answered OQs, tightens the memo, and writes the result back to the same file
- [ ] `--consolidate` is idempotent (same file + same in-file answers → byte-identical file)

### Acceptance test

- [ ] First end-to-end test on empleo.digital produces a usable memo (PRD Success Metrics row 5)

## Testing requirements

| Test                            | Approach                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Frontmatter lint**            | For each agent file, assert required fields present + `tools` matches exactly + `model` is opus/sonnet per seat + slug matches filename (incl. `community-pr-lead`)                                                                                                                                                                                                |
| **Body section lint**           | For each agent, assert the 5 required `##` headers in order                                                                                                                                                                                                                                                                                                        |
| **Hard-fail (3 variants)**      | Manual: (a) move `agent_docs/marketing/` away, (b) remove one file, (c) delete one `##` section — verify message                                                                                                                                                                                                                                                   |
| **Parallel dispatch**           | Inspect the Agent-tool call: assert single fan-out of 8, not 8 sequential                                                                                                                                                                                                                                                                                          |
| **Agent failure handling**      | Simulate one agent timing out; assert one retry occurs; on persistent failure, assert N-1 memo with `[FAILED:]` row and Brief-recap note                                                                                                                                                                                                                           |
| **Synthesizer reference file**  | Assert `reference/synthesizer-prompt.md` exists, is non-empty, and is referenced by `commands/marketing-board.md`                                                                                                                                                                                                                                                  |
| **Same-turn OQ generation**     | Inspect the synthesizer turn: assert memo + OQ section emitted in a single Claude response, not two separate calls                                                                                                                                                                                                                                                 |
| **OQ-contract shape lint**      | For 3 sample memos, assert OQ section has: (a) intro line, (b) summary table with `ID \| Question \| Status` columns, (c) one `### OQ-NN — <title>` detail block per row, (d) each detail block contains `**Question:**`, `**Why it matters:**`, `**Possible answers:**` (with `- [ ]` checkboxes), `**Status:**`. Matches prd-writer / tdd-writer output verbatim |
| **OQ relevance check**          | Manual: review each OQ in 3 sample memos; assert each would actually block a launch decision (not cosmetic, not already answered in brief / KB)                                                                                                                                                                                                                    |
| **H2 wrapper present**          | For 3 sample deliberations, assert each agent's report begins with `## <Seat> Report`                                                                                                                                                                                                                                                                              |
| **Memo template**               | Run 3 deliberations on different briefs; assert all 11 memo sections present in every output                                                                                                                                                                                                                                                                       |
| **Verdicts-table count**        | Assert table row count = 8 in every memo                                                                                                                                                                                                                                                                                                                           |
| **Attribution rule**            | Inspect memo: "Where the board disagrees" entries name seats; other sections do not                                                                                                                                                                                                                                                                                |
| **Consolidate idempotency**     | Run `--consolidate` twice on the same memo file + same in-file answers; diff = empty                                                                                                                                                                                                                                                                               |
| **Memo always saved**           | Run a deliberation; assert `marketing-plans/<slug>-<YYYY-MM-DD>.md` is created with no flag passed                                                                                                                                                                                                                                                                 |
| **Save filename collision**     | Run two deliberations on the same day; assert the second filename has a `-<HHMMSS>` suffix and the first file is untouched                                                                                                                                                                                                                                         |
| **Consolidate writes in place** | Answer OQs in a saved memo file, run `--consolidate`; assert the same file is updated (no new file created)                                                                                                                                                                                                                                                        |
| **Empleo acceptance**           | Manual end-to-end: bootstrap → deliberate → review; record qualitative pass/fail                                                                                                                                                                                                                                                                                   |
| **Reusability**                 | Run on a second SaaS without plugin modification; assert no errors                                                                                                                                                                                                                                                                                                 |

No automated test framework in v1 (per TDD 00 testing strategy). All tests are manual + lint-style assertions on file structure.

## Authorization

N/A — personal-use Claude Code plugin. See TDD 00.

## Related Documents

- **Master TDD**: [`00-marketing-board-master.md`](00-marketing-board-master.md)
- **Sibling TDD** (Bootstrap): [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md)
- **Parent PRD**: [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md)
- **Pattern**: [`plugins/ceo-board/commands/board.md`](../../../plugins/ceo-board/commands/board.md) (sibling implementation to mirror)
- **Subagent docs**: <https://code.claude.com/docs/en/sub-agents>

## Open Questions

*All questions resolved and integrated as of 2026-05-20.*

### Resolution log

| ID      | Question (short)                             | Resolution                                                                                                                                                                                                          | Integrated in                                                                                                                          |
| ------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| OQ-D-01 | OQ deep-analysis prompt placement            | **Same Claude turn** — synthesizer emits memo + OQ section inline as a single response                                                                                                                              | Body step sequence (step 5 + step 6 merged); synthesizer step S-11                                                                     |
| OQ-D-02 | Multi-memo disambiguation on `--consolidate` | **Most recently written memo file in `marketing-plans/`** when no path arg supplied. (Superseded by the always-save revision — memos are now files, not conversation-only; see PRD FR-20 / Architecture Decision 9) | Flag contracts table, `--consolidate` flag contract                                                                                    |
| OQ-D-03 | Agent report H2 wrapper                      | **`## <Seat> Report` H2 wrapper required** as first line of every agent's report                                                                                                                                    | "Per-seat output-format skeleton" intro paragraph; acceptance criterion                                                                |
| OQ-D-04 | Agent failure behavior                       | **Retry once; on persistent failure, proceed with N-1 + `[FAILED:]` row** in verdicts table + Brief-recap note + `[verify retry]` OQ callout                                                                        | New behavior spec, memo template note, acceptance criteria, testing requirements                                                       |
| OQ-D-05 | Synthesizer prompt location                  | **Reference file** at `plugins/marketing-board/reference/synthesizer-prompt.md`; loaded by `commands/marketing-board.md` at step 5                                                                                  | "Synthesizer prompt structure" section rewritten; new acceptance criteria; **cross-doc impact on TDD 00** (file structure tree update) |

### Cross-document propagation (applied)

- **From master TDD's OQ-M-01**: agent slug `community-pr` → **`community-pr-lead`** (per-seat configuration table updated)
- **To master TDD** (queued for v1.1): add `plugins/marketing-board/reference/synthesizer-prompt.md` to the plugin file structure tree (applied in this consolidation pass)
