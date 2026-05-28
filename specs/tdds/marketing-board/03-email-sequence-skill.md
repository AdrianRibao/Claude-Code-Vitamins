# TDD — Email Sequence Skill

| Field        | Value                                                                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Type         | Feature TDD                                                                                                                                      |
| Status       | v1.0 — Phase 0 decisions locked; Phase 2 OQs pending                                                                                             |
| Owner        | Adrián Ribao                                                                                                                                     |
| Created      | 2026-05-22                                                                                                                                       |
| Parent PRD   | [`specs/prds/marketing-board/01-email-sequence-skill.md`](../../prds/marketing-board/01-email-sequence-skill.md)                                 |
| Master TDD   | [`00-marketing-board-master.md`](00-marketing-board-master.md)                                                                                   |
| Sibling TDDs | [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md), [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md) |
| First test   | empleo.digital — drafts against `marketing-plans/empleo-digital-2026-05-22.md`                                                                   |
| Plugin host  | `plugins/marketing-board/` (additive — no changes to existing files)                                                                             |
| Invocation   | `/marketing-board:email-sequence`                                                                                                                |

______________________________________________________________________

## Overview

Specifies the contracts for the `/marketing-board:email-sequence` skill — a new child of the existing `marketing-board` plugin that consumes a saved Marketing Plan Memo and drafts ESP-paste-ready email sequences with subject / preheader / CTA variants rendered as `- [ ]` checkboxes. The founder picks variants in-file and runs `--consolidate` to bake the picks in.

Cross-cutting contracts (knowledge-base file shapes, marketplace conventions, naming) live in the [master TDD](00-marketing-board-master.md). The Marketing Plan Memo shape this skill parses is defined in [TDD 01](01-marketing-board-deliberation.md). The knowledge-base file shapes are defined in [TDD 00 master](00-marketing-board-master.md) and produced by [TDD 02 bootstrap](02-marketing-board-bootstrap.md).

## Scope

**In:**

- `plugins/marketing-board/skills/email-sequence/SKILL.md` — frontmatter + body contract
- Memo-parsing contract (LLM extraction in the synthesis turn)
- Knowledge-base reading contract (which sections from which files; `constraints.md` conditional)
- Output file shape — per-sequence file at `email-sequences/<slug>-<YYYY-MM-DD>.md`
- Per-email block shape — header, trigger, metric, subject + 2-3 `- [ ]` variants, preheader + 2 `- [ ]` variants, body (sequence-type-driven length), optional Visual block, CTA + 1-2 `- [ ]` variants
- Persona ask-back UX (batch upfront — one pause max per run)
- `--sequence <name>` and `--consolidate` flag contracts
- Conditional emissions: compliance block, visual block, sequence-level OQ block, freshness warning
- Hard-fail message contracts (verbatim strings)

**Out:**

- Knowledge-base file shapes (data contracts) → TDD 00 master
- Marketing Plan Memo shape → TDD 01 deliberation
- Bootstrap flow → TDD 02
- Sibling future production skills (video-script, lp-audit) — separate future TDDs
- ESP-specific export formats — v1.1
- Calling image-generation APIs — v1.1
- Automated test framework — manual testing only per the family convention (see TDD 00 testing strategy)

## File structure

```
plugins/marketing-board/skills/
└── email-sequence/
    └── SKILL.md                  # Single-file skill — no template, no scripts
```

No `templates/` subdirectory (output is dynamically generated per memo, not a static form). No `reference/` subdirectory (the body is small enough to live in `SKILL.md`). No `scripts/` (the skill is pure Markdown + LLM, no external code).

## Skill frontmatter contract

| Field                      | Value                                                                                                                                                                                                                                                           |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                     | `email-sequence`                                                                                                                                                                                                                                                |
| `description`              | "Draft ESP-paste-ready email sequences from a saved Marketing Plan Memo. Generates subject/preheader/CTA variants as `- [ ]` checkboxes for in-file picking; `--consolidate` bakes the picks in. Explicit invocation only — `/marketing-board:email-sequence`." |
| `disable-model-invocation` | `true`                                                                                                                                                                                                                                                          |
| `allowed-tools`            | `Read, Grep, Glob, Write, Edit`                                                                                                                                                                                                                                 |
| `argument-hint`            | `[<sequence-name>] [@<memo-path>] [--consolidate]`                                                                                                                                                                                                              |

No other frontmatter fields. No `model:` override (skill runs in the user's current model). No `hooks:`, `mcpServers:`, `memory:`.

## SKILL.md body — required sections

The body MUST contain these sections in this order:

| Section                    | Purpose                                                                                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `## When to use`           | "Run after a marketing-board deliberation has produced a saved memo. Drafts ESP-paste-ready email sequences with variants."                                   |
| `## Inputs`                | Names the memo file resolution rule + knowledge-base files read (always-read vs conditional)                                                                  |
| `## Hard-fail conditions`  | Lists each hard-fail trigger and its exact message string (verbatim per FR-03/04/05/21)                                                                       |
| `## Drafting flow`         | Step-by-step: locate memo → extract sequences (LLM) → persona judgment → batch ask-back if any ambiguous → draft → write files                                |
| `## Persona ask-back`      | The exact UX: collect all ambiguous-persona sequences, ask in one message, draft after reply (per AD-9 + Phase-0 decision)                                    |
| `## Consolidate flow`      | Step-by-step: locate file → parse checkbox state → apply picks + preserve prose edits → emit run summary with "still needs decisions" list → rewrite in place |
| `## Output file shape`     | Per-sequence file template structure: preamble, optional compliance block, per-email blocks, optional sequence-level OQ block                                 |
| `## Per-email block shape` | Detailed structural contract per FR-08: header, trigger, metric, subject variants, preheader variants, body, optional visual, CTA variants                    |
| `## Idempotency & safety`  | Same-day collision suffix, never-overwrite, consolidate idempotency, KB freshness warning                                                                     |

No other top-level sections in v1.

## Memo-parsing contract (LLM extraction)

Per Phase 0 decision: extraction happens in the **same synthesis turn** as drafting — no separate parser pass.

| Aspect                 | Spec                                                                                                                                                                                                                                                                                    |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Target heading         | `### Email sequences (outlines)` — LLM matches case-insensitively and tolerates minor variants (extra whitespace, alternate punctuation). If absent → hard-fail per FR-04                                                                                                               |
| Extracted shape        | Per sequence: `{name, trigger, target_metric, email_count, per_email_outlines[]}`. `per_email_outlines` is a list of 1-line descriptions (may be empty if the outline is thin — triggers FR-21 invention gate)                                                                          |
| Invention gate (FR-21) | When `per_email_outlines` is empty AND the outline supplies sequence name + trigger + target metric, the LLM infers per-email arcs and marks each with `<!-- INFERRED: outline was thin; review carefully -->`. When any of name/trigger/target metric is missing → hard-fail per FR-21 |
| Source of truth        | Memo only — the skill never proposes sequences absent from the outline (per AD-7)                                                                                                                                                                                                       |
| No separate parse      | The extraction prompt and the drafting prompt are in the same Claude turn — one round-trip                                                                                                                                                                                              |

## Knowledge-base reading contract

| File                                  | Sections read                                        | Required?   | Purpose                                                                                             |
| ------------------------------------- | ---------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------- |
| `agent_docs/marketing/product.md`     | `## Brand voice & tone`, `## Value propositions`     | Required    | Voice calibration; body content material                                                            |
| `agent_docs/marketing/audience.md`    | `## Personas`, `## Vocabulary`, `## Triggers to buy` | Required    | Persona scene; buyer terms; subject-line trigger material                                           |
| `agent_docs/marketing/business.md`    | `## Funnel state`                                    | Optional    | LP URL — substituted inline per FR-23 when present                                                  |
| `agent_docs/marketing/constraints.md` | `## Legal / compliance`                              | Conditional | Read always; compliance block emitted only when section names GDPR / CAN-SPAM / similar (per FR-24) |

If `product.md` or `audience.md` is missing OR lacks any required `##` section → hard-fail per FR-05 with the exact message `"Bootstrap your product context first: /marketing-board:bootstrap"`.

## Output file shape (locked)

One file per sequence at `email-sequences/<sequence-slug>-<YYYY-MM-DD>.md`. Same-day collision → `-<HHMMSS>` suffix (never overwrite).

```markdown
# <Sequence name> sequence — <product>

> Drafted from: marketing-plans/<product>-<YYYY-MM-DD>.md (per FR-17)

**Trigger:** <from memo>
**Target metric:** <from memo>
**Persona:** <name from audience.md>
**Voice notes:** <adjective list from product.md>

⚠️  Brand voice has been updated since this memo was generated; drafts use the current voice. Re-run /marketing-board for a consistent memo.
   <!-- emitted only when product.md mtime > memo mtime (per FR-22) -->

## Compliance check
   <!-- emitted only when constraints.md `## Legal / compliance` names GDPR / CAN-SPAM / similar (per FR-24, OQ-10) -->

- <constraint-specific compliance reminder>

## 01 — <email name>

<per-email block per the locked shape below>

## 02 — <email name>

<per-email block>

…

## Open Questions
   <!-- emitted only when the LLM identifies sequence-level execution-blocking decisions (per FR-25, OQ-05) -->

*Sequence-level decisions surfaced during drafting — resolve in this file, then run `--consolidate`.*

| ID    | Question                    | Status |
| ----- | --------------------------- | ------ |
| OQ-01 | <one-line question>          | Open   |

### OQ-01 — <Title>

**Question:** …

**Why it matters:** …

**Possible answers:**

- [ ] Option A …
- [ ] Option B …

**Status:** Open — recommend Option A because …
```

Conditional blocks emit ONLY when their trigger condition holds. An empty `## Compliance check` block is not emitted (no constraints triggered = block absent entirely).

## Per-email block shape (locked)

````markdown
## <NN> — <email name>

**Trigger:** <email-specific trigger from outline>
**Target metric:** <email-level or sequence-level metric>

**Subject (pick one):**

- [ ] <variant 1>
- [ ] <variant 2>
- [ ] <variant 3>      <!-- optional 3rd variant -->

**Preheader (pick one):**

- [ ] <variant 1>
- [ ] <variant 2>

**Body:**

<body prose at sequence-type-driven length — see length table below>

**Visual:**
   <!-- emitted only when the LLM judges a visual would strengthen this email (per FR-18, FR-19, OQ-11) -->

- **Prompt:** <tool-agnostic image-generation prompt; brand-calibrated; persona-aware scene; play-button overlay if video thumbnail>
- **Alt text:** <accessible description>
- **HTML embed:**

  ```html
  <a href="<VIDEO_URL>" target="_blank" rel="noopener">
    <img src="<THUMBNAIL_URL>" alt="..." width="600" style="display:block; max-width:100%; height:auto; border:0;" />
  </a>
````

**CTA (pick one):**

- [ ] \<variant 1>
- [ ] \<variant 2> <!-- optional 2nd variant -->

```

## Sequence-type → body length defaults table (per FR-08, OQ-02)

The LLM matches the sequence name against this lookup (case-insensitive substring) and uses the matched body-length range. Unrecognized sequence names fall back to 150-250.

| Match keyword(s)                                                | Body length (words) |
| --------------------------------------------------------------- | ------------------- |
| `welcome`, `onboarding`                                         | 100-180             |
| `activation`                                                    | 150-250             |
| `trial-to-paid`, `trial to paid`, `t2p`, `paid conversion`      | 200-350             |
| `win-back`, `winback`, `re-engagement`, `reengagement`, `dormant`| 100-180             |
| `nurture`, `education`                                          | 200-300             |
| (anything else)                                                 | 150-250             |

The lookup is intentionally narrow — common sequence-name patterns get tuned defaults; anything novel gets a sane fallback.

## Persona ask-back UX (per AD-9 + Phase-0 decision)

When the LLM judges multiple sequences with ambiguous persona, the skill emits a SINGLE message at the top of the turn listing all of them, then waits for one founder reply, then drafts all sequences in one synthesis pass.

Message shape:

```

Persona-ambiguous sequences detected — pick one for each:

- Activation — Carlos, Marta, or Luis?
- Trial-to-paid — Marta or Luis?

Reply with your picks (e.g. "Activation: Marta. Trial-to-paid: Luis."), and I'll draft everything.

```

Edge cases:

- No ambiguity → skill proceeds directly to drafting; no message
- Ambiguity in only one sequence → still uses this format (singular bullet)
- Founder reply doesn't cover all asked → skill re-asks the unanswered ones in one follow-up

## Flag contracts

| Invocation                                                  | Behavior                                                                                                                  |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `/marketing-board:email-sequence`                           | Default. Drafts ALL sequences in the most recently modified `marketing-plans/*.md` memo                                   |
| `/marketing-board:email-sequence <name>`                    | Positional `--sequence`. Drafts ONLY the named sequence; case-insensitive exact match against the memo's sequence-name list |
| `/marketing-board:email-sequence --sequence <name>`         | Explicit form of the positional. Same behavior                                                                            |
| `/marketing-board:email-sequence @<memo-path>`              | Uses the specified memo file instead of "most recent." Combinable with `<name>`                                          |
| `/marketing-board:email-sequence --consolidate`             | Consolidate mode. Operates on the most recently modified `email-sequences/*.md` file                                      |
| `/marketing-board:email-sequence --consolidate <name>`      | Consolidate mode targeting the named sequence's file (matches filename slug)                                              |
| `/marketing-board:email-sequence --consolidate @<path>`     | Consolidate mode targeting the specified file                                                                             |

Mutually exclusive: `--consolidate` cannot be combined with `<sequence-name>` AND `@<memo-path>` simultaneously (the `@<path>` for consolidate points to a sequence file, not a memo). The skill detects and aborts on conflicting args with: `"Conflicting arguments. Use one of: <name>, @<path>, or --consolidate [<name>|@<path>]."`

## Hard-fail message contracts (verbatim)

| ID         | Condition                                                          | Message (verbatim)                                                                                                                                                                       |
| ---------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HF-NO-MEMO | No memo file resolvable (default mode)                              | `"No memo found. Run /marketing-board <brief> first."`                                                                                                                                   |
| HF-NO-OUTLINE | Memo lacks parseable email-sequences section                     | `"Memo has no parseable '### Email sequences (outlines)' section. The Funnel Engineer seat must have produced an outline. Re-run /marketing-board to regenerate."`                       |
| HF-NO-KB   | `product.md` or `audience.md` missing or required `##` section absent | `"Bootstrap your product context first: /marketing-board:bootstrap"`                                                                                                                    |
| HF-THIN-OUTLINE | Sequence outline lacks sequence name + trigger + target metric  | `"Sequence '<name>' lacks the minimum context (trigger or target metric) needed to infer per-email content. Edit the memo to add this context, then re-run."`                            |
| HF-CONSOLIDATE-NOFILE | `--consolidate` invoked but no `email-sequences/*.md` file found | `"No email-sequence file found to consolidate. Run /marketing-board:email-sequence first."`                                                                                              |
| HF-CONFLICTING-ARGS | `<name>`, `@<path>`, and `--consolidate` combined illegally | `"Conflicting arguments. Use one of: <name>, @<path>, or --consolidate [<name>|@<path>]."`                                                                                              |

No partial-context fallback. No silent degradation. Inheriting marketing-board's hard-fail discipline (AD-6).

## Behavior specifications

### Happy path — full memo

**Given** `marketing-plans/empleo-digital-2026-05-22.md` exists with a Funnel Engineer outline listing 3 sequences (Welcome, Activation, Win-back)
**And** `agent_docs/marketing/{product,audience}.md` exist with all required `##` sections
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill extracts the 3 sequences from the memo (LLM in synthesis turn)
**And** drafts every email in each sequence with the locked per-email block shape
**And** writes `email-sequences/welcome-2026-05-22.md`, `email-sequences/activation-2026-05-22.md`, `email-sequences/win-back-2026-05-22.md`
**And** reports the saved paths plus a reminder to pick variants and run `--consolidate`

### Single-sequence drafting

**Given** the same memo state as above
**When** the founder runs `/marketing-board:email-sequence welcome`
**Then** only `email-sequences/welcome-2026-05-22.md` is written
**And** the other two sequences are not drafted (no files written for them)

### Persona ambiguity — batch ask-back

**Given** the memo lists 2 sequences with personas the LLM judges as ambiguous (Activation could be Carlos or Marta; Trial-to-paid could be Marta or Luis)
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill emits ONE message asking about both ambiguous sequences at the top of the turn
**And** waits for the founder's reply
**When** the founder replies "Activation: Marta. Trial-to-paid: Luis."
**Then** the skill drafts ALL sequences in one synthesis pass using the resolved personas
**And** writes the per-sequence files

### Missing memo

**Given** `marketing-plans/` does not exist (or is empty) and no `@<path>` is supplied
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill aborts with the HF-NO-MEMO message
**And** no files are written

### Memo lacks email-sequences section

**Given** a memo exists but the Funnel Engineer's `### Email sequences (outlines)` block is absent
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill aborts with the HF-NO-OUTLINE message
**And** no files are written

### Outline lacks per-email descriptions (FR-21 invention gate)

**Given** the memo's outline names a sequence + email count but supplies no per-email descriptions
**And** the outline DOES supply sequence name + trigger + target metric (the three required signals)
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill invents the per-email arc using the three signals as cues
**And** marks each inferred per-email block with `<!-- INFERRED: outline was thin; review carefully -->`
**And** writes the file normally

**Given** the outline lacks trigger OR target metric (one of the three required signals missing)
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill aborts with the HF-THIN-OUTLINE message naming the deficient sequence

### Same-day collision (never overwrite)

**Given** `email-sequences/welcome-2026-05-22.md` already exists (founder ran the skill earlier today and annotated the file)
**When** the founder runs `/marketing-board:email-sequence welcome` again on the same day
**Then** the new draft is written to `email-sequences/welcome-2026-05-22-<HHMMSS>.md`
**And** the original file is not modified

### Consolidate — happy path

**Given** `email-sequences/welcome-2026-05-22.md` exists with 4 emails, each with subject and CTA variants
**And** the founder ticked one `- [x]` subject variant + one CTA variant per email
**When** the founder runs `/marketing-board:email-sequence --consolidate`
**Then** the skill identifies the file (most-recent in `email-sequences/`)
**And** for each email: keeps only the ticked subject/CTA/preheader variants; removes the others
**And** preserves any prose edits the founder made to body content verbatim
**And** writes the consolidated content back to the same file in place
**And** running `--consolidate` again on the same file + same picks produces a byte-identical file

### Consolidate — no picks (warn, don't decide)

**Given** the same file but the founder ticked nothing
**When** the founder runs `--consolidate`
**Then** the skill keeps ALL variants for every email (consolidate is a no-op on variant content)
**And** the run summary lists every email under "still needs decisions"
**And** the file content is otherwise unchanged

### Consolidate — all picks (warn, don't decide)

**Given** the founder ticked every subject and CTA variant in one email (oops)
**When** the founder runs `--consolidate`
**Then** the skill keeps all ticked variants for that email (no narrowing)
**And** the run summary includes that email under "still needs decisions" with a hint about over-ticking
**And** other emails consolidate normally

### Freshness warning (FR-22)

**Given** `agent_docs/marketing/product.md` has a more recent mtime than the source memo
**When** the founder runs `/marketing-board:email-sequence`
**Then** the skill drafts using the current `product.md` voice
**And** the run summary includes the freshness warning verbatim per FR-22
**And** each saved sequence file includes the same warning in the preamble area

### Conditional compliance block (FR-24)

**Given** `constraints.md`'s `## Legal / compliance` section names GDPR
**When** the founder runs `/marketing-board:email-sequence`
**Then** every saved sequence file includes a `## Compliance check` block at the top (after the preamble, before email 01)
**And** the block names the specific constraint (GDPR) and the implied requirements (unsubscribe link, sender address)

**Given** `constraints.md`'s `## Legal / compliance` section says "None" or names only non-email-relevant constraints
**When** the founder runs `/marketing-board:email-sequence`
**Then** no `## Compliance check` block is emitted in any sequence file

### Conditional visual block (FR-18, OQ-11)

**Given** the LLM judges (during the synthesis turn) that email 02 of the Welcome sequence references a demo video that warrants a thumbnail
**When** the founder runs `/marketing-board:email-sequence`
**Then** email 02's block includes a `**Visual:**` section with prompt + alt-text + HTML embed
**And** other emails in the same sequence (where the LLM judged no visual was needed) have no `**Visual:**` section

### Conditional sequence-level OQ block (FR-25)

**Given** the LLM identifies an execution-blocking decision for a sequence (e.g., "the trigger 'signup verified' isn't defined in `product.md` — which event in your funnel?")
**When** the founder runs `/marketing-board:email-sequence`
**Then** the sequence's file includes a `## Open Questions` block at the bottom in the locked render shape
**And** the block contains one or more `### OQ-NN — <title>` detail blocks with checkboxed possible answers

**Given** the LLM identifies no execution-blocking decisions for a sequence
**When** the founder runs `/marketing-board:email-sequence`
**Then** no `## Open Questions` block is emitted in that sequence's file

## Acceptance criteria

### Skill structure

- [ ] `plugins/marketing-board/skills/email-sequence/SKILL.md` exists
- [ ] Frontmatter has `name: email-sequence`, `disable-model-invocation: true`, `allowed-tools: Read, Grep, Glob, Write, Edit`, `argument-hint: [<sequence-name>] [@<memo-path>] [--consolidate]`
- [ ] Body contains the 9 required `##` sections in the locked order

### Memo parsing

- [ ] LLM extraction happens in the same synthesis turn as drafting (one Claude round-trip, not two)
- [ ] Skill matches `### Email sequences (outlines)` heading case-insensitively with reasonable tolerance for whitespace / punctuation variants
- [ ] Skill never proposes sequences absent from the memo's outline

### Knowledge-base reads

- [ ] Skill reads `product.md` `## Brand voice & tone` and `## Value propositions`
- [ ] Skill reads `audience.md` `## Personas`, `## Vocabulary`, `## Triggers to buy`
- [ ] Skill reads `business.md` `## Funnel state` when present (optional)
- [ ] Skill reads `constraints.md` `## Legal / compliance` always; emits compliance block only when GDPR/CAN-SPAM/similar named
- [ ] Skill does NOT read `competition.md` or `distribution.md`

### Drafting

- [ ] Default invocation drafts every sequence in the memo's outline (no skipped sequences)
- [ ] `--sequence <name>` (or positional) scopes to one sequence; case-insensitive name match
- [ ] Every drafted email has the 7 required block elements: header, Trigger, Target metric, Subject (2-3 `- [ ]` variants), Preheader (2 `- [ ]` variants), Body, CTA (1-2 `- [ ]` variants)
- [ ] Body length matches the sequence-type lookup (Welcome/Win-back 100-180; Activation 150-250; Trial-to-paid 200-350; fallback 150-250)
- [ ] Visual block is emitted ONLY when the LLM judges it would strengthen the email (no keyword matching)
- [ ] Drafts follow the memo's language (LLM implicit detection)
- [ ] `## Drafted from:` line at the file top names the source memo path

### Persona ask-back UX

- [ ] When zero sequences are persona-ambiguous, the skill proceeds directly to drafting (no message)
- [ ] When ≥1 sequences are persona-ambiguous, the skill emits ONE batch message at the top of the turn listing all of them
- [ ] After the founder's reply, the skill drafts all sequences in one synthesis pass
- [ ] If the founder's reply doesn't cover all asked sequences, the skill re-asks only the unanswered ones

### File writing

- [ ] Output dir is `email-sequences/` in the working directory; created if absent
- [ ] Filename is `<sequence-slug>-<YYYY-MM-DD>.md`; slug is the sequence name kebab-cased
- [ ] Same-day collision → `-<HHMMSS>` suffix; existing files are never overwritten
- [ ] No binary files are written (no image files in v1)
- [ ] LP URL from `business.md`'s `## Funnel state` is substituted inline when present; obvious placeholder (`<https://YOUR-…>`) otherwise

### Conditional blocks

- [ ] Compliance block appears at the file top (after preamble, before email 01) ONLY when `constraints.md` `## Legal / compliance` names GDPR/CAN-SPAM/similar
- [ ] Visual block appears in a per-email block ONLY when the LLM judges a visual would strengthen that specific email
- [ ] Sequence-level OQ block appears at the file bottom ONLY when the LLM identifies execution-blocking decisions for that sequence
- [ ] Freshness warning appears in the file preamble AND run summary ONLY when `product.md` mtime > source memo mtime

### Consolidate

- [ ] `--consolidate` (no args) operates on the most recently modified `email-sequences/*.md` file
- [ ] `--consolidate <name>` targets the named sequence's file (slug match)
- [ ] `--consolidate @<path>` targets the specified file
- [ ] Ticked variants kept; unticked variants removed
- [ ] Founder's prose edits to body content preserved verbatim
- [ ] No picks (or all picks) per email → keep all + add to "still needs decisions" list in the run summary; file otherwise unchanged
- [ ] Running `--consolidate` twice on the same file + same in-file picks produces a byte-identical file

### Hard-fail messaging

- [ ] HF-NO-MEMO message displayed verbatim when no memo can be resolved
- [ ] HF-NO-OUTLINE displayed verbatim when memo lacks the email-sequences block
- [ ] HF-NO-KB displayed verbatim when `product.md` or `audience.md` is missing / lacks required `##` section
- [ ] HF-THIN-OUTLINE displayed verbatim when a sequence outline lacks trigger or target metric
- [ ] HF-CONSOLIDATE-NOFILE displayed verbatim when `--consolidate` finds no file
- [ ] HF-CONFLICTING-ARGS displayed verbatim when args conflict

### Acceptance test

- [ ] First end-to-end test on `marketing-plans/empleo-digital-2026-05-22.md` produces a usable sequence file for at least one sequence (PRD Success Metrics row 6)

## Testing requirements

| Test                          | Approach                                                                                                                                                                                |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Frontmatter lint**          | Assert SKILL.md frontmatter has exactly the 5 required fields with the exact values from the contract table                                                                            |
| **Body section lint**         | Assert the 9 required `##` headers appear in the locked order                                                                                                                          |
| **Hard-fail messaging**       | Manual: trigger each of the 6 hard-fail conditions (missing memo, missing outline, missing KB, thin outline, consolidate-no-file, conflicting args); assert each message appears verbatim |
| **Locked output shape**       | Run against the empleo memo; assert the saved file's structure matches the locked output-file template (preamble, optional blocks, per-email blocks)                                  |
| **Per-email block shape**     | For 3 drafted emails, assert each contains exactly: `## NN — <name>`, `**Trigger:**`, `**Target metric:**`, `**Subject (pick one):**` block with 2-3 `- [ ]` lines, `**Preheader (pick one):**` block with 2 `- [ ]` lines, `**Body:**` prose, `**CTA (pick one):**` block with 1-2 `- [ ]` lines |
| **Body length defaults**      | For one Welcome-named and one Trial-to-paid-named sequence, assert body lengths fall in 100-180 and 200-350 word ranges respectively                                                  |
| **Single-sequence scope**     | Run with positional `welcome` arg; assert only `email-sequences/welcome-<date>.md` is written                                                                                          |
| **Persona ask-back batch**    | Simulate two persona-ambiguous sequences; assert single batch message; assert drafting proceeds after one reply                                                                       |
| **Same-day collision**        | Run twice on the same day; assert second filename has `-<HHMMSS>` suffix and the first file is unchanged                                                                              |
| **Consolidate idempotency**   | Pre-create a file with ticked variants; run `--consolidate` twice; diff between the two outputs is empty                                                                              |
| **Consolidate no-picks**      | Pre-create a file with zero ticked variants; run `--consolidate`; assert "still needs decisions" appears in the run summary; assert file content unchanged                            |
| **Conditional compliance**    | Set `constraints.md` `## Legal / compliance` to (a) name GDPR, (b) say "None"; run skill in each; assert compliance block present in (a), absent in (b)                              |
| **Conditional visual**        | Manual review of 5 drafted emails; assert at most one or two carry a `**Visual:**` block (those where the LLM judged it added value)                                                  |
| **Conditional sequence OQ**   | Manual review; assert a `## Open Questions` block appears only when the drafting surfaced an execution-blocker                                                                        |
| **Freshness warning**         | Touch `product.md` to a newer mtime than the memo; run skill; assert warning appears in run summary and preamble                                                                      |
| **LP URL substitution**       | Set `business.md` `## Funnel state` to contain `https://empleo.digital`; run; assert that URL appears inline in body / CTA where the email references the landing page                |
| **Empleo end-to-end**         | Manual: run against `marketing-plans/empleo-digital-2026-05-22.md`; review one sequence; assert it would be usable with light editing                                                  |

No automated test framework in v1 (per TDD 00 testing strategy). All tests are manual + lint-style assertions on file structure.

## Authorization

**N/A** — same as the rest of the marketing-board plugin. Personal Claude Code skill in the user's working directory. No multi-user authorization. See [TDD 00 master](00-marketing-board-master.md).

## Related Documents

- **Parent PRD**: [`specs/prds/marketing-board/01-email-sequence-skill.md`](../../prds/marketing-board/01-email-sequence-skill.md) — defines goals, non-goals, FRs (25), success metrics, and 10 architecture decisions
- **Master TDD**: [`00-marketing-board-master.md`](00-marketing-board-master.md) — cross-cutting contracts (file layout, naming, knowledge-base file shapes, plugin metadata)
- **Sibling TDD** (Deliberation): [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md) — defines the Marketing Plan Memo template + the Funnel Engineer's `### Email sequences (outlines)` block this skill parses
- **Sibling TDD** (Bootstrap): [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md) — defines the knowledge-base file shapes this skill reads; precedent for skill structure + `--consolidate` flag discipline + hybrid-engine pattern (which we deliberately did NOT use here per Phase 0 Q3)
- **Master PRD**: [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md) — parent of the parent PRD; locks the marketing-board plugin's v1 scope and architecture decisions

## Open Questions

*Generated via deep analysis after synthesis — resolve before implementation. Answer in this file (tick the checkboxes, add notes), then run `/specs-plugin:tdd-writer --consolidate @specs/tdds/marketing-board/03-email-sequence-skill.md` to fold answers in.*

| ID    | Question                                                                                                | Status                              |
| ----- | ------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| OQ-01 | Consolidated-variant render shape — single line vs preserved block vs single line + comment trail       | Open — recommend Option 1           |
| OQ-02 | Sequence-slug normalization rules (accents, special chars, multi-word names)                            | Open — recommend Option 3           |
| OQ-03 | `--consolidate <name>` disambiguation when multiple files match the slug                                | Open — recommend Option 2           |
| OQ-04 | Persona ambiguity threshold — operationalizing AD-9's "LLM judges as ambiguous"                         | Open — recommend Option 1           |
| OQ-05 | Non-English sequence names — how the body-length lookup handles them                                    | Open — recommend Option 1           |
| OQ-06 | LP URL extraction heuristic from `business.md`'s `## Funnel state` prose                                | Open — recommend Option 4           |
| OQ-07 | Extraction validation — preventing silent miss of a sequence by the LLM                                  | Open — recommend Option 4           |
| OQ-08 | `--consolidate` semantics for the sequence-level `## Open Questions` block (per FR-25)                  | Open — recommend Option 1           |

### OQ-01 — Consolidated-variant render shape

**Question:** After `--consolidate` resolves variant picks, how should the kept variant be rendered? E.g., the `**Subject (pick one):**` block of three checkboxed variants becomes what exactly in the post-consolidate file?

**Why it matters:** Every consolidated email has three variant blocks (Subject, Preheader, CTA) that need a render decision. Inconsistent rendering complicates the parse on re-consolidate. The chosen shape sets a stable contract that affects every consolidated email the skill ever produces.

**Possible answers:**

- [ ] Single line: `**Subject:** <chosen variant>`. Removes the checkbox infrastructure entirely. Re-consolidate becomes trivially idempotent (no variants left to pick).
- [ ] Preserve block with one ticked variant: `**Subject (pick one):**\n\n- [x] <chosen variant>`. Keeps the format consistent across pre- and post-consolidate but is visually weird (a single-option "pick one").
- [ ] Single line + comment trail: `**Subject:** <chosen>\n\n<!-- alternatives removed during consolidate: <list> -->`. Preserves the rejected alternatives inline as comments for the founder's reference.
- [ ] Option 1 plus a `## Consolidation log` footer at the file bottom listing what was kept for each email. Clean visible record without inline noise.

**Status:** Open — recommend Option 1 (single line). Clean; git history preserves the alternatives if the founder needs them; re-consolidate is trivially idempotent.

### OQ-02 — Sequence-slug normalization rules

**Question:** What's the exact normalization rule for converting a sequence name to its filename slug? Accent handling? Special characters? Multi-word names with parentheses?

**Why it matters:** Filenames must be safe across platforms (Linux/macOS/Windows). The rule determines what `--sequence trial-to-paid` matches against, what file gets written for "Activación early" or "Welcome (week 1)", and whether the founder ever has to debug encoding issues.

**Possible answers:**

- [ ] Lowercase + replace whitespace/dots/underscores with hyphens + strip non-alphanumeric (Unicode-aware). Preserves accented letters as Unicode in filenames: `Activación` → `activación`. Locale-friendly, less portable.
- [ ] Same as Option 1 but strip-accents first (Unicode NFD-then-strip-marks): `Activación` → `activacion`. ASCII-friendly slugs.
- [ ] Strict ASCII-safe: lowercase + strip accents + collapse any run of non-`[a-z0-9]` characters into a single hyphen + trim leading/trailing hyphens. `Welcome (week 1)` → `welcome-week-1`; `Activación early` → `activacion-early`. Most portable.
- [ ] Preserve case and accents; only replace whitespace with hyphens. Most readable but breaks portability and case-sensitive matching.

**Status:** Open — recommend Option 3 (strict ASCII-safe + collapse non-alphanumerics). Portable across filesystems; deterministic; the founder doesn't need to debug encoding issues; matches the convention the rest of the family uses for slugs.

### OQ-03 — `--consolidate <name>` disambiguation when multiple files match

**Question:** When the founder runs `/marketing-board:email-sequence --consolidate welcome` and `email-sequences/` contains multiple files matching the welcome slug (e.g., from different days of iteration), which file does the skill consolidate?

**Why it matters:** Common scenario after a few days of iteration on the same sequence — the founder has `welcome-2026-05-22.md` (annotated heavily) and `welcome-2026-05-29.md` (re-drafted later). Silently consolidating the wrong file overwrites annotations; refusing to pick is friction.

**Possible answers:**

- [ ] Most recently modified file (mtime). Silent default.
- [ ] Most recently modified file, but the run summary prints the chosen path + how to override with `@<path>`. Defaults sensibly; transparent about the choice.
- [ ] List matching files + ask the founder which to consolidate. Interactive; safer; one extra round-trip.
- [ ] Most recent date in the filename (parsed `<slug>-<YYYY-MM-DD>.md`). Deterministic by name; ignores mtime; potentially surprising if the founder edited an older file recently.

**Status:** Open — recommend Option 2 (mtime + visible report). The most-recently-touched file is almost always the one the founder is working on; the visible report keeps the choice transparent; `@<path>` available for explicit override.

### OQ-04 — Persona ambiguity threshold

**Question:** AD-9 says the skill pauses for ask-back when the LLM's persona confidence is "low." What signal makes the LLM call a sequence's persona ambiguous?

**Why it matters:** Setting the threshold too low triggers ask-back on every sequence (annoying UX bloat). Too high produces silent best-guesses on genuinely ambiguous cases. Operationalizes AD-9's "low confidence" — the difference between a slick UX and a frustrating one.

**Possible answers:**

- [ ] Black-box LLM judgment — the LLM decides per sequence with no explicit criteria. Trust the model's reasoning; the batch-upfront ask-back UX bounds the cost of false positives.
- [ ] Explicit numeric criterion in the SKILL.md: "Ambiguous when ≥2 personas have ≥40% relevance to the trigger AND the gap between top-1 and top-2 is <20 percentage points." Quantifies the LLM's reasoning; potentially over-constraining.
- [ ] Explicit lexical criterion: "Ambiguous when the audience file has ≥2 personas AND the Funnel Engineer's trigger doesn't lexically reference any persona's day-in-the-life or JTBD." Simple test the LLM can apply consistently.
- [ ] Founder-overridable: default to LLM judgment; `--strict-persona` flag forces ask-back on every sequence (founder always picks).

**Status:** Open — recommend Option 1 (black-box LLM judgment). The model is good at this kind of judgment; explicit numeric thresholds mis-fire; the batch ask-back UX (one pause max per run) bounds the cost when the LLM gets it wrong. Keep the criterion implicit; trust the model.

### OQ-05 — Non-English sequence names + body-length lookup

**Question:** The body-length lookup table (FR-08) uses English keywords (welcome / activation / trial-to-paid / win-back / etc.). How does it match non-English sequence names like "Bienvenida" or "Activación" or "Reenganche"?

**Why it matters:** Empleo.digital is Spanish — its Funnel Engineer will likely name sequences in Spanish. Mis-matching (or always falling back to the 150-250 default) loses the per-sequence length tuning the PRD locked in via OQ-02. First acceptance test depends on this.

**Possible answers:**

- [ ] LLM translates each sequence name to English-keyword space internally during the synthesis turn, then matches the table. Single source of truth (table stays English-canonical); multilingual memos work without code changes.
- [ ] Extend the lookup table with bilingual keywords (es: bienvenida → welcome; activación → activation; reenganche → win-back; conversión → trial-to-paid; etc.). Predictable; deterministic; needs a table extension per supported language.
- [ ] LLM judges body length per sequence using its own knowledge of email norms (no table). Drops the table entirely; trusts the model; loses the locked length-per-type contract.
- [ ] All non-English sequence names fall back to default 150-250; no per-type tuning for non-English memos. Simple; loses the FR-08 advantage for non-English memos.

**Status:** Open — recommend Option 1 (LLM translates internally). Preserves the locked length-per-type contract from FR-08; the table stays English-canonical; multilingual memos work without code changes.

### OQ-06 — LP URL extraction heuristic

**Question:** `business.md`'s `## Funnel state` is prose — TDD 00 specifies the body as "LP URL, signup flow description (prose), known conversion rates if any." How does the skill find the LP URL within that prose for FR-23 substitution?

**Why it matters:** Wrong URL extracted (or no extraction when one is present) breaks the FR-23 promise of "LP URL substituted inline when present." Affects every email body / CTA that references the landing page across every sequence.

**Possible answers:**

- [ ] Regex match for the first URL pattern (`https?://[^\s]+`) in the section. Simple and fast; can mis-match when the section mentions multiple URLs (LP + competitor URL + tracking URL).
- [ ] LLM extracts the URL with context awareness ("which of these URLs is the landing page?"). Smarter; adds an LLM step.
- [ ] Require a structured `**LP URL:**` line in `business.md`'s `## Funnel state` (TDD 00 amendment + bootstrap-skill amendment). Deterministic but extends the family.
- [ ] LLM extracts during the existing synthesis turn — no separate extraction step, the model is already reading the full section for drafting. One less moving part.

**Status:** Open — recommend Option 4 (LLM extracts during synthesis). The model is already reading `business.md` for the drafting pass; one less moving part than a separate regex or extraction step; no TDD 00 amendment needed.

### OQ-07 — Extraction validation (silent-miss prevention)

**Question:** If the LLM extracts only 2 sequences from a memo that actually outlines 3, the skill silently produces 2 drafts. How does the skill prevent that silent failure?

**Why it matters:** The founder might not notice the missing sequence — they get a usable output for 2 sequences and assume the third didn't exist in the outline. Hours of email work missed, no error message. Real failure mode for an LLM-driven extraction.

**Possible answers:**

- [ ] Run summary lists the extracted sequence names + email counts; the founder cross-checks visually against the memo. Lightweight; no extra LLM call.
- [ ] Two-pass extraction: first prompt asks the LLM to COUNT sequences only; second prompt extracts and drafts; skill compares counts and warns on mismatch. Defensive but adds a round-trip + cost.
- [ ] Accept the silent miss — the founder reviews the output anyway, and the failure mode is rare enough not to engineer for. Lean.
- [ ] Run summary + explicit "if a sequence is missing, re-run with `--sequence <name>` and the original memo" reminder. Visible, founder-actionable, no extra LLM cost.

**Status:** Open — recommend Option 4 (run summary + re-run guidance). Visible, founder-actionable, no extra LLM cost; the re-run guidance turns a silent failure into a self-correctable one.

### OQ-08 — `--consolidate` semantics for the sequence-level OQ block

**Question:** When a drafted sequence file has a `## Open Questions` block (per FR-25) and the founder ticks answers and runs `--consolidate`, what does the skill do with the OQ block? Mirror marketing-board's memo-consolidate behavior, or leave it alone?

**Why it matters:** The drafted file's OQ block is rendered in marketing-board's locked OQ shape (intro line + summary table + per-OQ detail blocks). Marketing-board's `--consolidate` folds answers into the memo's relevant sections and marks OQs Resolved. The TDD doesn't specify whether email-sequence's consolidate inherits the same behavior.

**Possible answers:**

- [ ] Mirror marketing-board's consolidate — fold answers into the relevant per-email blocks where applicable; mark resolved OQs `Resolved (YYYY-MM-DD)`; collapse the OQ block to a resolution log when all are resolved.
- [ ] Leave the OQ block alone — `--consolidate` only handles variant picks; OQ-block answers are the founder's responsibility to fold into the prose manually.
- [ ] Mark answered OQs as Resolved in the block but don't auto-fold answers anywhere; the founder edits prose separately.
- [ ] Strip the OQ block entirely after consolidate — the answers have been picked; the questions no longer block execution; the file gets cleaner.

**Status:** Open — recommend Option 1 (mirror marketing-board). The founder already knows that pattern from marketing-board's memos; reusing it minimizes the mental model; the OQ-block render shape matches marketing-board's exactly, so the consolidation logic is structurally identical.
```
