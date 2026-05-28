# TDD — Email Sequence Skill

| Field        | Value                                                                                                                                                          |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type         | Feature TDD                                                                                                                                                    |
| Status       | v1.4 — implemented 2026-05-28; reconciled to no-Bash limits (count suffix; filename-date recency; FR-22 deferred); heading matcher made level-agnostic after TDD 01 added `## Email sequences (outlines)` to the memo |
| Owner        | Adrián Ribao                                                                                                                                                   |
| Created      | 2026-05-22                                                                                                                                                     |
| Updated      | 2026-05-28 — `--consolidate` folded 8 OQ resolutions; implementation pass reconciled mtime/clock contracts to the no-Bash tool grant                           |
| Parent PRD   | [`specs/prds/marketing-board/01-email-sequence-skill.md`](../../prds/marketing-board/01-email-sequence-skill.md)                                               |
| Master TDD   | [`00-marketing-board-master.md`](00-marketing-board-master.md)                                                                                                 |
| Sibling TDDs | [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md), [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md)               |
| First test   | empleo.digital — drafts against `marketing-plans/empleo-digital-2026-05-22.md`                                                                                 |
| Plugin host  | `plugins/marketing-board/` (additive — no changes to existing files)                                                                                           |
| Invocation   | `/marketing-board:email-sequence`                                                                                                                              |

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
- Conditional emissions: compliance block, visual block, sequence-level OQ block
- Hard-fail message contracts (verbatim strings)

**Out:**

- Knowledge-base file shapes (data contracts) → TDD 00 master
- Marketing Plan Memo shape → TDD 01 deliberation
- Bootstrap flow → TDD 02
- Sibling future production skills (video-script, lp-audit) — separate future TDDs
- ESP-specific export formats — v1.1
- Calling image-generation APIs — v1.1
- **Freshness warning (FR-22)** — v1.1. Warning when `product.md` changed after the memo was generated needs cross-directory file-mtime comparison, which a no-Bash skill cannot do reliably (see [Capability constraints](#capability-constraints-no-bash-skill)). Deferred until the skill can read mtimes.
- Automated test framework — manual testing only per the family convention (see TDD 00 testing strategy)

## Capability constraints (no-Bash skill)

`allowed-tools` is `Read, Grep, Glob, Write, Edit` — deliberately no Bash. This bounds what the skill can observe about the filesystem, and several contracts below follow from it:

| Constraint                                  | Consequence                                                                                                                                                             |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| No wall-clock access                        | Same-day collision suffix is a **count-based `-<N>`** (`-2`, `-3`, …), not a timestamp. The skill `Glob`s existing `<slug>-<date>*.md` and picks the next free integer. |
| No file-mtime access                        | "Most recent" memo / sequence file is resolved by the **date encoded in the filename** (`<slug>-<YYYY-MM-DD>.md`, ties broken by highest `-<N>`), not by mtime.         |
| Cross-directory mtime comparison impossible | The **freshness warning (FR-22) is deferred to v1.1** — `product.md` carries no filename date, so its recency relative to the memo is unobservable without Bash.        |

`Glob` does sort by mtime, but only **within one pattern**, capped at 100 results, and exposes ordering — not timestamps. That supports single-directory "latest" only when filenames lack a usable date; here filenames always carry the date, so date-from-filename is the deterministic, clock-free signal used throughout.

## Cascading amendments required (per OQ-06)

OQ-06's resolution (frontmatter `lp_url` key in `business.md` — refined post-consolidation; see resolution-log note) commits sibling-doc amendments **before** this skill ships. This establishes YAML frontmatter as the family's structured-metadata convention for the six `agent_docs/marketing/*.md` knowledge-base files (optional per file; used when there are non-prose values like URLs, IDs, locale codes, color hex):

| Doc                                                 | Amendment                                                                                                                                                                                       |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [TDD 00 master](00-marketing-board-master.md)       | Adds a "Frontmatter convention" section for KB files; `business.md` shape spec adds an optional YAML frontmatter block with `lp_url: <url>` as the first canonical structured field             |
| [TDD 02 bootstrap](02-marketing-board-bootstrap.md) | `INTERVIEW.md` Section 4 (Business) gains a dedicated LP URL question (single-line); `:bootstrap --consolidate` writes the answer to `business.md`'s frontmatter; section question count: 6 → 7 |

Example resulting `business.md`:

```markdown
---
lp_url: https://empleo.digital
---

# Business

## Pricing tiers

[…]

## Funnel state

[prose about the funnel]
```

The email-sequence skill parses `business.md`'s YAML frontmatter and reads `lp_url` deterministically (no LLM inference for the URL). Falls back to the obvious placeholder `<https://YOUR-LP-URL>` when the frontmatter is absent or `lp_url` is missing.

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
| `## Idempotency & safety`  | Same-day collision suffix (count-based), never-overwrite, consolidate idempotency, filename-date recency                                                      |

No other top-level sections in v1.

## Memo-parsing contract (LLM extraction)

Per Phase 0 decision: extraction happens in the **same synthesis turn** as drafting — no separate parser pass.

| Aspect                 | Spec                                                                                                                                                                                                                                                                                    |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Target heading         | `Email sequences (outlines)` heading at any level (`## ` in the memo, `### ` in the raw Funnel Engineer report) — LLM matches the heading text case-insensitively and tolerates minor variants (whitespace, punctuation, heading level). If absent → hard-fail per FR-04                  |
| Extracted shape        | Per sequence: `{name, trigger, target_metric, email_count, per_email_outlines[]}`. `per_email_outlines` is a list of 1-line descriptions (may be empty if the outline is thin — triggers FR-21 invention gate)                                                                          |
| Invention gate (FR-21) | When `per_email_outlines` is empty AND the outline supplies sequence name + trigger + target metric, the LLM infers per-email arcs and marks each with `<!-- INFERRED: outline was thin; review carefully -->`. When any of name/trigger/target metric is missing → hard-fail per FR-21 |
| Source of truth        | Memo only — the skill never proposes sequences absent from the outline (per AD-7)                                                                                                                                                                                                       |
| No separate parse      | The extraction prompt and the drafting prompt are in the same Claude turn — one round-trip                                                                                                                                                                                              |

## Knowledge-base reading contract

| File                                  | Sections read                                        | Required?   | Purpose                                                                                                                                                                                                                    |
| ------------------------------------- | ---------------------------------------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_docs/marketing/product.md`     | `## Brand voice & tone`, `## Value propositions`     | Required    | Voice calibration; body content material                                                                                                                                                                                   |
| `agent_docs/marketing/audience.md`    | `## Personas`, `## Vocabulary`, `## Triggers to buy` | Required    | Persona scene; buyer terms; subject-line trigger material                                                                                                                                                                  |
| `agent_docs/marketing/business.md`    | YAML frontmatter `lp_url` key (per OQ-06 refined)    | Optional    | LP URL — substituted inline per FR-23 when present; deterministic extraction via frontmatter parse (no LLM inference); falls back to `<https://YOUR-LP-URL>` placeholder when frontmatter is absent or `lp_url` is missing |
| `agent_docs/marketing/constraints.md` | `## Legal / compliance`                              | Conditional | Read always; compliance block emitted only when section names GDPR / CAN-SPAM / similar (per FR-24)                                                                                                                        |

If `product.md` or `audience.md` is missing OR lacks any required `##` section → hard-fail per FR-05 with the exact message `"Bootstrap your product context first: /marketing-board:bootstrap"`.

## Output file shape (locked)

One file per sequence at `email-sequences/<sequence-slug>-<YYYY-MM-DD>.md`. Same-day collision → count-based `-<N>` suffix (`-2`, `-3`, …; never overwrite).

```markdown
# <Sequence name> sequence — <product>

> Drafted from: marketing-plans/<product>-<YYYY-MM-DD>.md (per FR-17)

**Trigger:** <from memo>
**Target metric:** <from memo>
**Persona:** <name from audience.md>
**Voice notes:** <adjective list from product.md>

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

**Non-English sequence names (per OQ-05)**: the LLM internally translates each sequence name to English-keyword space during the synthesis turn before matching the lookup. Spanish examples: `Bienvenida` → matches `welcome`; `Activación` → matches `activation`; `Reenganche` → matches `re-engagement`; `Conversión` → matches `trial-to-paid`. Translation is implicit (no separate prompt step); the table stays English-canonical; multilingual memos work without code changes.

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

**Ambiguity threshold (per OQ-04)**: the LLM judges ambiguity with no explicit numeric or lexical criterion — black-box judgment. Trust the model; the batch ask-back UX (one pause max per run) bounds the cost of false positives. The SKILL.md body does NOT enumerate when to call a persona ambiguous beyond "the LLM judges multiple personas plausibly fit the sequence's trigger / target metric."

## Flag contracts

| Invocation                                                  | Behavior                                                                                                                  |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `/marketing-board:email-sequence`                           | Default. Drafts ALL sequences in the latest-dated `marketing-plans/*.md` memo (date from filename)                        |
| `/marketing-board:email-sequence <name>`                    | Positional `--sequence`. Drafts ONLY the named sequence; case-insensitive exact match against the memo's sequence-name list |
| `/marketing-board:email-sequence --sequence <name>`         | Explicit form of the positional. Same behavior                                                                            |
| `/marketing-board:email-sequence @<memo-path>`              | Uses the specified memo file instead of "most recent." Combinable with `<name>`                                          |
| `/marketing-board:email-sequence --consolidate`             | Consolidate mode. Operates on the latest-dated `email-sequences/*.md` file (date from filename)                           |
| `/marketing-board:email-sequence --consolidate <name>`      | Consolidate mode targeting the named sequence's file (matches filename slug)                                              |
| `/marketing-board:email-sequence --consolidate @<path>`     | Consolidate mode targeting the specified file                                                                             |

Mutually exclusive: `--consolidate` cannot be combined with `<sequence-name>` AND `@<memo-path>` simultaneously (the `@<path>` for consolidate points to a sequence file, not a memo). The skill detects and aborts on conflicting args with: `"Conflicting arguments. Use one of: <name>, @<path>, or --consolidate [<name>|@<path>]."`

## Hard-fail message contracts (verbatim)

| ID         | Condition                                                          | Message (verbatim)                                                                                                                                                                       |
| ---------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HF-NO-MEMO | No memo file resolvable (default mode)                              | `"No memo found. Run /marketing-board <brief> first."`                                                                                                                                   |
| HF-NO-OUTLINE | Memo lacks parseable email-sequences section                     | `"Memo has no parseable 'Email sequences (outlines)' section. The Funnel Engineer seat must have produced an outline. Re-run /marketing-board to regenerate."`                           |
| HF-NO-KB   | `product.md` or `audience.md` missing or required `##` section absent | `"Bootstrap your product context first: /marketing-board:bootstrap"`                                                                                                                    |
| HF-THIN-OUTLINE | Sequence outline lacks sequence name + trigger + target metric  | `"Sequence '<name>' lacks the minimum context (trigger or target metric) needed to infer per-email content. Edit the memo to add this context, then re-run."`                            |
| HF-CONSOLIDATE-NOFILE | `--consolidate` invoked but no `email-sequences/*.md` file found | `"No email-sequence file found to consolidate. Run /marketing-board:email-sequence first."`                                                                                              |
| HF-CONFLICTING-ARGS | `<name>`, `@<path>`, and `--consolidate` combined illegally | `"Conflicting arguments. Use one of: <name>, @<path>, or --consolidate [<name>|@<path>]."`                                                                                              |

No partial-context fallback. No silent degradation. Inheriting marketing-board's hard-fail discipline (AD-6).

## Sequence-slug normalization rules (per OQ-02)

The sequence name → filename slug conversion is Unicode-aware (preserves accented characters):

1. Apply Unicode NFC normalization
2. Lowercase (Unicode-aware — `İ` → `i̇` per Unicode case-folding)
3. Replace whitespace, dots, and underscores with single hyphens
4. Strip characters that are not Unicode letters, Unicode digits, or hyphens (parentheses, brackets, slashes, etc. removed)
5. Collapse multiple consecutive hyphens into one
6. Trim leading and trailing hyphens

| Sequence name        | Slug                |
| -------------------- | ------------------- |
| `Welcome`            | `welcome`           |
| `Bienvenida`         | `bienvenida`        |
| `Activación early`   | `activación-early`  |
| `Welcome (week 1)`   | `welcome-week-1`    |
| `Trial-to-paid`      | `trial-to-paid`     |
| `Reenganche / dormant` | `reenganche-dormant` |

Accents are preserved as Unicode codepoints in the filename. This requires a filesystem with Unicode filename support (ext4, APFS, NTFS — all modern targets). The skill does NOT strip-accents-to-ASCII.

## Consolidate flow contract (per OQ-01, OQ-03, OQ-08)

### File targeting (per OQ-03)

When `--consolidate` runs without `@<path>`:

1. Scope candidates to `email-sequences/*.md`
2. If `<name>` is supplied, filter to filenames whose slug matches `<name>` (case-insensitive)
3. Pick the candidate with the latest date in its filename (ties broken by highest `-<N>` suffix)
4. **Print the chosen path in the run summary** along with the hint: `"To consolidate a different file, pass @<path>."`

If zero candidates → hard-fail `HF-CONSOLIDATE-NOFILE`. If multiple candidates match a `<name>` filter, the visible report shows the founder which file was picked (no silent default).

### Variant rendering after consolidate (per OQ-01)

For each email block, the three variant blocks (Subject, Preheader, CTA) collapse from a checkbox list to a single line based on the founder's `- [x]` picks:

| Pre-consolidate                                                                 | Post-consolidate (single pick)                |
| ------------------------------------------------------------------------------- | --------------------------------------------- |
| `**Subject (pick one):**\n\n- [x] First pick\n- [ ] Alternative\n- [ ] Other`   | `**Subject:** First pick`                     |
| `**Preheader (pick one):**\n\n- [x] First pick\n- [ ] Alternative`              | `**Preheader:** First pick`                   |
| `**CTA (pick one):**\n\n- [x] First pick\n- [ ] Alternative`                    | `**CTA:** First pick`                         |

The unticked alternatives are removed from the file. Git history preserves them if the founder needs to recover them. Re-running `--consolidate` on a post-consolidate file is a byte-identical no-op (idempotency).

### Sequence-level OQ-block consolidate (per OQ-08)

When the drafted sequence file contains a `## Open Questions` block (per FR-25) and the founder has ticked answers, `--consolidate` mirrors marketing-board's memo consolidate behavior:

1. Fold the picked OQ answer into the relevant per-email block where applicable (e.g., if OQ resolved which trigger to use, update affected email blocks' `**Trigger:**` line)
2. Mark each resolved OQ in the summary table with `Resolved (YYYY-MM-DD) — <one-line answer>`
3. Drop the `### OQ-NN — <title>` detail blocks for resolved questions; keep only the summary table row
4. When ALL OQs in the block are resolved, replace the `## Open Questions` block with a compact one-line resolution log: `*All N sequence-level decisions resolved on YYYY-MM-DD and integrated above.*`

Unresolved OQs (no `[x]` ticked) remain visible with full detail blocks.

## Run summary contract (per OQ-07)

After every drafting run (not `--consolidate`), the skill emits a run summary to the conversation containing:

| Item                              | Purpose                                                                                                                  |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Extracted sequence names + counts | Founder cross-check: `"Extracted 3 sequences from memo: Welcome (4 emails), Activation (5), Win-back (3)."`              |
| Saved file paths                  | One bullet per sequence: `"- email-sequences/welcome-2026-05-22.md"`                                                     |
| Re-run guidance for misses        | Verbatim: `"If a sequence in the memo is missing here, re-run with /marketing-board:email-sequence <name> @<memo-path>."` |
| Variant-pick reminder             | `"Tick - [x] on one variant per Subject / Preheader / CTA block, then run /marketing-board:email-sequence --consolidate."` |

(Freshness warning row removed — FR-22 deferred to v1.1; see [Capability constraints](#capability-constraints-no-bash-skill).)

For `--consolidate` runs, the summary instead contains: chosen-file path (per OQ-03), count of variants collapsed, count of OQs resolved (if any), list of emails added to "still needs decisions" (no-picks or all-picks per FR-15 / FR-16).

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

**Given** a memo exists but its `## Email sequences (outlines)` section is absent (e.g. the Funnel Engineer seat failed, so the synthesizer omitted it)
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
**Then** the new draft is written to `email-sequences/welcome-2026-05-22-2.md` (count-based `-<N>` suffix)
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

### Freshness warning (FR-22) — deferred to v1.1

Not implemented in v1. Surfacing a warning when `product.md` changed after the memo was generated requires cross-directory file-mtime comparison, which a no-Bash skill cannot do reliably (see [Capability constraints](#capability-constraints-no-bash-skill)). When the skill gains a way to read mtimes, restore: draft using the current `product.md` voice, and emit the warning in both the run summary and each saved file's preamble.

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

### Non-English sequence name (per OQ-05)

**Given** the memo's outline lists a sequence named `Bienvenida`
**When** the founder runs `/marketing-board:email-sequence`
**Then** the LLM internally translates `Bienvenida` to `welcome` for the body-length lookup
**And** body length falls in the 100-180 word range (welcome bucket)
**And** the saved file is `email-sequences/bienvenida-2026-05-22.md` (slug preserves the original name, per OQ-02)
**And** body prose is in Spanish (per FR-09 implicit language detection)

### LP URL substitution (per OQ-06)

**Given** `business.md`'s YAML frontmatter contains `lp_url: https://empleo.digital`
**When** the founder runs `/marketing-board:email-sequence`
**Then** every body / CTA reference to "the landing page" in drafted emails is substituted with `https://empleo.digital` inline
**And** the substitution is deterministic (frontmatter parse, no LLM inference for the URL itself)

**Given** `business.md` has no YAML frontmatter (or has frontmatter but no `lp_url` key)
**When** the founder runs `/marketing-board:email-sequence`
**Then** body / CTA references to the landing page use the placeholder `<https://YOUR-LP-URL>`
**And** no hard-fail is triggered (LP URL is optional)

### Unicode-aware slug (per OQ-02)

**Given** the memo's outline lists a sequence named `Activación early`
**When** the founder runs `/marketing-board:email-sequence`
**Then** the saved file is `email-sequences/activación-early-2026-05-22.md` (the accented character is preserved)
**And** the file opens correctly on Linux ext4, macOS APFS, and Windows NTFS

### Consolidate disambiguation — multiple files match the slug (per OQ-03)

**Given** `email-sequences/welcome-2026-05-22.md` AND `email-sequences/welcome-2026-05-29.md` both exist
**When** the founder runs `/marketing-board:email-sequence --consolidate welcome`
**Then** the skill picks `welcome-2026-05-29.md` (latest date in filename)
**And** the run summary contains the chosen path verbatim and the hint `"To consolidate a different file, pass @<path>."`
**And** `welcome-2026-05-22.md` is not modified

### Consolidate — variant rendering collapses to single line (per OQ-01)

**Given** `email-sequences/welcome-2026-05-22.md` exists with email 01 having Subject block of 3 variants, second variant ticked
**When** the founder runs `/marketing-board:email-sequence --consolidate`
**Then** email 01's Subject section is rewritten as `**Subject:** <second variant text>`
**And** the unticked variants are removed from the file
**And** the same transformation applies to Preheader and CTA blocks
**And** re-running `--consolidate` on the resulting file produces a byte-identical output

### Consolidate — sequence-level OQ block resolved (per OQ-08)

**Given** `email-sequences/welcome-2026-05-22.md` has a `## Open Questions` block with one OQ
**And** the founder ticked one possible answer
**When** the founder runs `/marketing-board:email-sequence --consolidate`
**Then** the relevant per-email block(s) are updated with the picked answer's content (e.g., `**Trigger:**` line updated when the OQ resolved which trigger to use)
**And** the OQ's summary-table row shows `Resolved (YYYY-MM-DD) — <one-line answer>`
**And** the OQ's `### OQ-NN — <title>` detail block is removed
**And** when ALL OQs in the block are resolved, the entire `## Open Questions` block is replaced with `*All N sequence-level decisions resolved on YYYY-MM-DD and integrated above.*`

### Run summary contents (per OQ-07)

**Given** the happy-path scenario (3 sequences extracted)
**When** the founder runs `/marketing-board:email-sequence`
**Then** the run summary contains all 4 mandated items: extracted sequence names + email counts; saved file paths; re-run guidance for misses (verbatim); variant-pick reminder
**And** the re-run guidance is exactly `"If a sequence in the memo is missing here, re-run with /marketing-board:email-sequence <name> @<memo-path>."`

## Acceptance criteria

### Skill structure

- [ ] `plugins/marketing-board/skills/email-sequence/SKILL.md` exists
- [ ] Frontmatter has `name: email-sequence`, `disable-model-invocation: true`, `allowed-tools: Read, Grep, Glob, Write, Edit`, `argument-hint: [<sequence-name>] [@<memo-path>] [--consolidate]`
- [ ] Body contains the 9 required `##` sections in the locked order

### Memo parsing

- [ ] LLM extraction happens in the same synthesis turn as drafting (one Claude round-trip, not two)
- [ ] Skill matches the `Email sequences (outlines)` heading at any level (`##` or `###`) case-insensitively, with reasonable tolerance for whitespace / punctuation variants
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
- [ ] Non-English sequence names are internally translated to English-keyword space before the body-length lookup (per OQ-05); the table stays English-canonical
- [ ] Visual block is emitted ONLY when the LLM judges it would strengthen the email (no keyword matching)
- [ ] Drafts follow the memo's language (LLM implicit detection)
- [ ] `## Drafted from:` line at the file top names the source memo path

### Run summary (per OQ-07)

- [ ] Drafting-mode run summary contains: extracted sequence names + email counts; saved file paths (one bullet each); re-run guidance for misses; variant-pick reminder
- [ ] Re-run guidance is verbatim `"If a sequence in the memo is missing here, re-run with /marketing-board:email-sequence <name> @<memo-path>."`
- [ ] Consolidate-mode run summary contains: chosen-file path (per OQ-03); count of variants collapsed; count of OQs resolved (if any); list of emails added to "still needs decisions"

### Persona ask-back UX

- [ ] When zero sequences are persona-ambiguous, the skill proceeds directly to drafting (no message)
- [ ] When ≥1 sequences are persona-ambiguous, the skill emits ONE batch message at the top of the turn listing all of them
- [ ] After the founder's reply, the skill drafts all sequences in one synthesis pass
- [ ] If the founder's reply doesn't cover all asked sequences, the skill re-asks only the unanswered ones
- [ ] Persona ambiguity uses black-box LLM judgment (per OQ-04) — no explicit numeric or lexical threshold encoded in SKILL.md

### File writing

- [ ] Output dir is `email-sequences/` in the working directory; created if absent
- [ ] Filename is `<sequence-slug>-<YYYY-MM-DD>.md`; slug follows the Unicode-aware normalization rule (per OQ-02): NFC + lowercase + replace whitespace/dots/underscores with `-` + strip non-alphanumeric (Unicode-aware) + collapse consecutive `-` + trim
- [ ] Accented characters in sequence names are preserved in filenames (e.g., `Activación` → `activación`)
- [ ] Same-day collision → count-based `-<N>` suffix (`-2`, `-3`, …); existing files are never overwritten
- [ ] No binary files are written (no image files in v1)
- [ ] LP URL is extracted deterministically from `business.md`'s YAML frontmatter `lp_url` key (per OQ-06 refined); falls back to `<https://YOUR-LP-URL>` placeholder when the frontmatter is absent or `lp_url` is missing — no LLM inference for URL extraction

### Conditional blocks

- [ ] Compliance block appears at the file top (after preamble, before email 01) ONLY when `constraints.md` `## Legal / compliance` names GDPR/CAN-SPAM/similar
- [ ] Visual block appears in a per-email block ONLY when the LLM judges a visual would strengthen that specific email
- [ ] Sequence-level OQ block appears at the file bottom ONLY when the LLM identifies execution-blocking decisions for that sequence
- [ ] Freshness warning — deferred to v1.1 (no-Bash skill cannot compare mtimes; see Capability constraints). Not asserted in v1.

### Consolidate

- [ ] `--consolidate` (no args) operates on the latest-dated `email-sequences/*.md` file (date from filename)
- [ ] `--consolidate <name>` targets the named sequence's file (slug match)
- [ ] `--consolidate @<path>` targets the specified file
- [ ] When multiple files match the `<name>` filter, the latest filename date wins (ties → highest `-<N>`) and the chosen path is printed in the run summary along with the `@<path>` override hint (per OQ-03)
- [ ] Ticked variants kept; unticked variants removed
- [ ] After consolidate, each variant block collapses to a single line: `**Subject:** <pick>`, `**Preheader:** <pick>`, `**CTA:** <pick>` (per OQ-01)
- [ ] Founder's prose edits to body content preserved verbatim
- [ ] No picks (or all picks) per email → keep all + add to "still needs decisions" list in the run summary; file otherwise unchanged
- [ ] Running `--consolidate` twice on the same file + same in-file picks produces a byte-identical file (idempotency)
- [ ] Sequence-level OQ blocks (per FR-25) consolidate by mirroring marketing-board's memo-consolidate behavior (per OQ-08): fold answers into per-email blocks, mark resolved OQs `Resolved (YYYY-MM-DD)`, drop resolved detail blocks, collapse block to one-line log when fully resolved

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
| **Same-day collision**        | Run twice on the same day; assert second filename has a count-based `-<N>` suffix (`-2`) and the first file is unchanged                                                              |
| **Consolidate idempotency**   | Pre-create a file with ticked variants; run `--consolidate` twice; diff between the two outputs is empty                                                                              |
| **Consolidate no-picks**      | Pre-create a file with zero ticked variants; run `--consolidate`; assert "still needs decisions" appears in the run summary; assert file content unchanged                            |
| **Conditional compliance**    | Set `constraints.md` `## Legal / compliance` to (a) name GDPR, (b) say "None"; run skill in each; assert compliance block present in (a), absent in (b)                              |
| **Conditional visual**        | Manual review of 5 drafted emails; assert at most one or two carry a `**Visual:**` block (those where the LLM judged it added value)                                                  |
| **Conditional sequence OQ**   | Manual review; assert a `## Open Questions` block appears only when the drafting surfaced an execution-blocker                                                                        |
| **Freshness warning**         | Deferred to v1.1 — not tested in v1 (no-Bash skill cannot compare mtimes; see Capability constraints)                                                                                |
| **LP URL substitution**       | Set `business.md` frontmatter to `lp_url: https://empleo.digital`; run; assert that URL appears inline in body / CTA where the email references the landing page                       |
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

*All 8 questions resolved on 2026-05-28 and integrated into the body sections above. This TDD is ready for implementation pending the cascading amendments to TDD 00 + TDD 02 (see the `## Cascading amendments required` callout at the top of this file).*

| ID    | Resolution                                                                                                                                       |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| OQ-01 | **Resolved (2026-05-28) — Option 1.** Variant blocks collapse to a single line post-consolidate. See `## Consolidate flow contract`.             |
| OQ-02 | **Resolved (2026-05-28) — Option 1.** Unicode-aware slug; accents preserved. See `## Sequence-slug normalization rules`.                          |
| OQ-03 | **Resolved (2026-05-28; refined at implementation).** Latest filename date wins (ties → highest `-<N>`); chosen path + override hint printed in the run summary. Refined from "mtime" to "filename date" because a no-Bash skill cannot observe mtimes. See `## Consolidate flow contract` + `## Capability constraints`. |
| OQ-04 | **Resolved (2026-05-28) — Option 1.** Black-box LLM judgment; no explicit numeric or lexical threshold. See `## Persona ask-back UX`.            |
| OQ-05 | **Resolved (2026-05-28) — Option 1.** LLM internally translates non-English sequence names before the lookup. See body-length defaults section.  |
| OQ-06 | **Resolved (2026-05-28; refined 2026-05-28).** Frontmatter `lp_url` key in `business.md` (refined from "structured body line" to YAML frontmatter — establishes the family's metadata convention). Cascading amendments to TDD 00 + TDD 02 + bootstrap skill required. |
| OQ-07 | **Resolved (2026-05-28) — Option 4.** Run summary lists extracted sequences + verbatim re-run guidance. See `## Run summary contract`.            |
| OQ-08 | **Resolved (2026-05-28) — Option 1.** Sequence-level OQ blocks mirror marketing-board's consolidate. See `## Consolidate flow contract`.         |
```
