# TDD — Marketing Board: Bootstrap

| Field       | Value                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------- |
| Type        | Feature TDD                                                                                                          |
| Status      | v1.0 — consolidated, OQs resolved                                                                                    |
| Owner       | Adrián Ribao                                                                                                         |
| Created     | 2026-05-20                                                                                                           |
| Parent TDD  | [`00-marketing-board-master.md`](00-marketing-board-master.md)                                                       |
| Parent PRD  | [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md) |
| Sibling TDD | [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md)                                           |

______________________________________________________________________

## Overview

Specifies the contracts for the **bootstrap flow**: the `/marketing-board:bootstrap` skill, the `INTERVIEW.md` scaffold structure, and the consolidation logic that produces the six knowledge-base files. Knowledge-base file shapes (the data contracts agents consume) live in the master TDD. Deliberation (what agents do with these files) lives in TDD 01.

## Scope

**In:**

- `skills/bootstrap/SKILL.md` — `/marketing-board:bootstrap` skill
- INTERVIEW.md schema: section structure + per-question render format + per-question intent table (locked via OQ-B-01)
- `--reset` and `--consolidate` flag contracts
- INTERVIEW.md → six knowledge-base files transformation (hybrid parser + LLM polish, per OQ-B-02)

**Out (deferred to v1.1):**

- `--lang <code>` flag and LLM runtime translation (per OQ-B-06 — v1 ships English-only; the *architecture* for translation is captured in master TDD's OQ-M-03 resolution and ready to wire in when v1.1 lands)

**Out:**

- Knowledge-base file shapes (data contracts) → TDD 00
- Deliberation flow → TDD 01

## Skill file — `skills/bootstrap/SKILL.md`

### Frontmatter

| Field                      | Value                                                                                                                                                                                                      |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                     | `marketing-board-bootstrap`                                                                                                                                                                                |
| `description`              | "Scaffold the marketing-board knowledge base (`agent_docs/marketing/`) for this project. Creates INTERVIEW.md; `--consolidate` produces 6 files. Explicit invocation only — `/marketing-board:bootstrap`." |
| `disable-model-invocation` | `true` (slash command only; never auto-fires)                                                                                                                                                              |

### Skill body — required sections

The SKILL.md body MUST instruct the agent on:

| Section                   | Purpose                                                                      |
| ------------------------- | ---------------------------------------------------------------------------- |
| `## When to use`          | "Run once per SaaS project before the first `/marketing-board` deliberation" |
| `## Flag handling`        | Parse `$ARGUMENTS` for `--reset` and `--consolidate` (no `--lang` in v1)     |
| `## Scaffold flow`        | Steps to create `agent_docs/marketing/INTERVIEW.md`                          |
| `## Consolidate flow`     | Steps to read INTERVIEW.md and produce the six target files                  |
| `## Idempotency & safety` | Abort/reset rules                                                            |

Supporting files: `skills/bootstrap/templates/interview.md` (single English source; per OQ-M-03 architecture + OQ-B-06 v1 scope). v1.1 will add `--lang <code>` flag and LLM-driven runtime translation.

## Flag contracts

| Invocation                                 | Pre-condition                              | Action                                                                                                                                      |
| ------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `/marketing-board:bootstrap`               | `agent_docs/marketing/` does **not** exist | Scaffold `agent_docs/marketing/INTERVIEW.md` (English) and the directory                                                                    |
| `/marketing-board:bootstrap`               | `agent_docs/marketing/` already exists     | **Abort** with: `"agent_docs/marketing/ already exists. Use --reset to overwrite INTERVIEW.md, or edit it directly and run --consolidate."` |
| `/marketing-board:bootstrap --lang <code>` | (any)                                      | **Abort** with: `"--lang is not available in v1. Bootstrap ships English-only; non-English support arrives in v1.1."` (per OQ-B-06)         |
| `/marketing-board:bootstrap --reset`       | (any)                                      | Overwrite `INTERVIEW.md` with fresh scaffold (English); **do NOT touch** the six target files                                               |
| `/marketing-board:bootstrap --consolidate` | `INTERVIEW.md` exists                      | Parse INTERVIEW.md; write/overwrite the six target files                                                                                    |
| `/marketing-board:bootstrap --consolidate` | `INTERVIEW.md` doesn't exist               | Abort with: `"INTERVIEW.md not found. Run /marketing-board:bootstrap first."`                                                               |

### Flag combination rules

| Combination             | Allowed | Behavior                                                 |
| ----------------------- | ------- | -------------------------------------------------------- |
| `--reset --consolidate` | No      | Abort: contradictory intent; user picks one              |
| Any flag + `--lang`     | No      | Abort: `--lang` is a v1.1 feature (see flag table above) |

## INTERVIEW.md schema

### Top-of-file preamble

The scaffolded INTERVIEW.md MUST begin with this preamble (English in v1):

| Element                    | Content                                                                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Title                      | `# Marketing-Board Interview — <product name>`                                                                                                                                       |
| Generated-by line          | `> Generated 2026-05-20 by /marketing-board:bootstrap (lang: en)`                                                                                                                    |
| One-paragraph instructions | "Fill the questions below thoughtfully. You can run `/marketing-board:bootstrap --consolidate` whenever; it's idempotent. Skip non-applicable questions with `[skipped: <reason>]`." |
| TOC                        | Bullet list linking to the six section headers                                                                                                                                       |

### Section structure

Six sections, in this order:

| #   | Section heading      | Maps to           | Q count (≈)                                   |
| --- | -------------------- | ----------------- | --------------------------------------------- |
| 1   | `## 1. Product`      | `product.md`      | 5                                             |
| 2   | `## 2. Audience`     | `audience.md`     | 8                                             |
| 3   | `## 3. Competition`  | `competition.md`  | 6                                             |
| 4   | `## 4. Business`     | `business.md`     | 7 (was 6; +1 LP URL per TDD 03 OQ-06 refined) |
| 5   | `## 5. Distribution` | `distribution.md` | 6                                             |
| 6   | `## 6. Constraints`  | `constraints.md`  | 5                                             |

Total: ~36 questions across the six sections.

### Section header structure

Each section opens with:

| Element                           | Purpose                                                                        |
| --------------------------------- | ------------------------------------------------------------------------------ |
| Section heading (`## N. <Topic>`) | Section identifier                                                             |
| "What this powers" line           | 1 sentence naming which agents read the resulting file                         |
| "Ask Claude to interview me" hint | Verbatim text the user can paste to switch to guided fill (see template below) |
| First question                    | Starts directly after, no nested heading                                       |

### "Ask Claude to interview me" hint — exact format

```markdown
> 💡 **Prefer guided fill?** Paste this to Claude:
> *"Interview me on the [Section Name] section — ask one question at a time and update INTERVIEW.md after each answer."*
```

(v1.1, when `--lang` ships, will translate this hint via LLM at runtime per OQ-M-03.)

### Per-question render format (locked)

Every question MUST render as:

```markdown
### Q<N.M>: <Question title>

**Why this matters:** <one line of rationale>

**Example:** *<one or two concrete worked examples in italics>*

**Detail level:** <one of: one sentence | one paragraph | bullet list | table>

**Your answer:**

<answer container — see "Answer container shape" below>

`[skip]` ← if not applicable, change this to `[skipped: <one-line reason>]`
```

Numbering: `Q1.1`, `Q1.2`, ..., `Q2.1`, etc. (Section-major numbering for stability across translations and for traceability to the **Question intent tables** below.)

### Question intent tables (locked per OQ-B-01 — Option 3)

The TDD locks each question's **intent** (what it asks for + answer type) but **not** the literal wording. The English template (`templates/interview.md`) owns the natural-language phrasing; v1.1 LLM translations re-render the wording per language. These tables are the testable contract.

#### Section 1: Product (5 questions)

| Q-ID | Intent                                                                                                         | Type                     |
| ---- | -------------------------------------------------------------------------------------------------------------- | ------------------------ |
| Q1.1 | Describe what the product does in 1-2 sentences                                                                | free-text                |
| Q1.2 | Pick the product category (with "Other" allowed)                                                               | single-choice            |
| Q1.3 | List 3-5 value propositions, each with a one-line proof point                                                  | bullet list              |
| Q1.4 | Draft a positioning statement using the "For [audience], [product] is the [category] that [unique value]" form | free-text                |
| Q1.5 | Select 3-5 voice/tone adjectives **and** add one example sentence in voice                                     | multi-select + free-text |

#### Section 2: Audience (8 questions)

| Q-ID | Intent                                                                                   | Type        |
| ---- | ---------------------------------------------------------------------------------------- | ----------- |
| Q2.1 | Define the ICP — segment, role, company size, geography, sophistication                  | bullet list |
| Q2.2 | Persona 1 — name, JTBD (one line), top 3 pains, day-in-the-life (2-3 sentences)          | free-text   |
| Q2.3 | Persona 2 — same shape as Q2.2                                                           | free-text   |
| Q2.4 | Persona 3 — same shape as Q2.2 (skip if fewer than 3 personas)                           | free-text   |
| Q2.5 | Anti-personas — 2-4 segments who are explicitly NOT the target, each with a one-line why | bullet list |
| Q2.6 | Attention map — channel, platform, signal strength (high/med/low) for each row           | table       |
| Q2.7 | Vocabulary — two columns: terms-they-use vs. terms-to-avoid                              | table       |
| Q2.8 | Triggers to buy — 3-5 events that prompt purchase consideration                          | bullet list |

#### Section 3: Competition (6 questions)

| Q-ID | Intent                                                                                           | Type        |
| ---- | ------------------------------------------------------------------------------------------------ | ----------- |
| Q3.1 | Name 3-5 alternatives (incl. status quo / DIY), each with a one-line description                 | bullet list |
| Q3.2 | How-we-differ table — alternative \| our advantage \| their advantage                            | table       |
| Q3.3 | Category conventions worth **inheriting**                                                        | bullet list |
| Q3.4 | Category conventions worth **breaking**, each with a one-line why                                | bullet list |
| Q3.5 | Competitive blind spots — what are competitors NOT addressing that you are?                      | bullet list |
| Q3.6 | Failed predecessors — companies that tried this approach and died; what killed them (best guess) | bullet list |

#### Section 4: Business (7 questions; Q4.3 added per TDD 03 OQ-06 refined)

| Q-ID | Intent                                                                                    | Type                                                                              |
| ---- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Q4.1 | Pricing tiers — table: tier, price, key inclusions, target persona                        | table                                                                             |
| Q4.2 | Primary pricing model (freemium / trial / paid-only / flat-fee / usage-based / undecided) | single-choice                                                                     |
| Q4.3 | Landing page URL — single URL                                                             | free-text (single-line; consolidate writes to `business.md` frontmatter `lp_url`) |
| Q4.4 | Funnel state — signup flow description + known conversion rates (or "none yet")           | free-text                                                                         |
| Q4.5 | Budget envelope — total + per-month allocation if known; "TBD" allowed                    | free-text                                                                         |
| Q4.6 | Timeline — target launch date or window + runway in months                                | free-text                                                                         |
| Q4.7 | Geographic focus — primary markets (countries / language regions) + locale code           | bullet list                                                                       |

#### Section 5: Distribution (6 questions)

| Q-ID | Intent                                                                                              | Type         |
| ---- | --------------------------------------------------------------------------------------------------- | ------------ |
| Q5.1 | Owned channels — bullet list with platform + current audience size                                  | bullet list  |
| Q5.2 | Content inventory — table: asset type \| count \| recency                                           | table        |
| Q5.3 | Founder communities — communities the founder is already active in                                  | bullet list  |
| Q5.4 | Aspirational influencers — table: name \| platform \| audience \| relevance                         | table        |
| Q5.5 | Past PR — bullet list of past mentions; "None" allowed                                              | bullet list  |
| Q5.6 | Launchable platforms — which launch surfaces matter for this product (PH, HN, Reddit, niche Slacks) | multi-select |

#### Section 6: Constraints (5 questions)

| Q-ID | Intent                                                                                              | Type                      |
| ---- | --------------------------------------------------------------------------------------------------- | ------------------------- |
| Q6.1 | Legal / compliance constraints (GDPR, sector regs, etc.); "None" allowed                            | bullet list               |
| Q6.2 | Founder risk tolerance — single choice: conservative / moderate / aggressive + 1-sentence rationale | single-choice + free-text |
| Q6.3 | Past failed experiments; "None" allowed                                                             | bullet list               |
| Q6.4 | Unfair advantages — 1-5 items, specific not generic                                                 | bullet list               |
| Q6.5 | Regulatory landmines — category-specific risks; "None" allowed                                      | bullet list               |

**Total: 36 questions.** Template wording can evolve freely as long as each Q-ID covers the intent in this table. The acceptance criterion is intent-based, not literal-wording-based.

### Answer container shapes (3 variants)

| Question type | Answer container                                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------------------- |
| Free-text     | Single fenced block `> _Your answer here..._` (the user replaces the italicized placeholder)                |
| Single-choice | Checkbox list — user changes **one** `[ ]` to `[x]`                                                         |
| Multi-select  | Checkbox list — user changes **any number** of `[ ]` to `[x]`. Section header notes "select all that apply" |

### Example: a free-text question (English template)

```markdown
### Q1.1: What does your product do?

**Why this matters:** Every agent needs the product baseline. The Ethnographer maps this to buyer pain; the Storyteller positions against alternatives.

**Example:** *"empleo.digital is a Spanish-language job board that matches blue-collar workers with verified employers in their city, reducing the friction of job-hunting from weeks to days."* (Good: specific audience, specific outcome, time dimension.) *Bad: "A platform that empowers users to achieve their career goals."* (Vague, generic, jargon.)

**Detail level:** one to two sentences

**Your answer:**

> _Your answer here..._

`[skip]` ← if not applicable, change this to `[skipped: <one-line reason>]`
```

### Example: a multi-select question (English template)

```markdown
### Q1.5: Brand voice & tone — select all that apply

**Why this matters:** Storyteller and Producer thread this voice through every asset.

**Example:** *Stripe uses precise + technical + understated. Notion uses friendly + playful + clear. Pick the 3-5 closest to yours.*

**Detail level:** check 3-5 below; add one example sentence in your voice in the free-text field at the end

- [ ] Friendly
- [ ] Authoritative
- [ ] Playful
- [ ] Technical / precise
- [ ] Warm / conversational
- [ ] Understated
- [ ] Bold / declarative
- [ ] Empathetic
- [ ] Direct
- [ ] Inspirational

**Example sentence in your voice:**

> _One sentence as your product would say it..._

`[skip]` ← if not applicable, change this to `[skipped: <one-line reason>]`
```

### Example: a single-choice question

```markdown
### Q4.2: What's your primary pricing model?

**Why this matters:** Channel Strategist and Funnel Engineer design entirely different funnels for freemium vs. paid-only.

**Example:** *Notion = freemium (free tier + paid teams). Linear = free trial only (no permanent free tier). Basecamp = flat fee.*

**Detail level:** select one

- [ ] Freemium (permanent free tier + paid)
- [ ] Free trial only (e.g. 14 days, then paid)
- [ ] Paid-only from day one
- [ ] Flat-fee per organization
- [ ] Usage-based / metered
- [ ] Not yet decided

`[skip]` ← if not applicable, change this to `[skipped: <one-line reason>]`
```

## INTERVIEW.md → knowledge-base files transformation

`--consolidate` performs this transformation. Mapping per section:

| INTERVIEW.md section | Output file       | Section ownership                            |
| -------------------- | ----------------- | -------------------------------------------- |
| 1. Product           | `product.md`      | All five `product.md` required sections      |
| 2. Audience          | `audience.md`     | All six `audience.md` required sections      |
| 3. Competition       | `competition.md`  | All four `competition.md` required sections  |
| 4. Business          | `business.md`     | All five `business.md` required sections     |
| 5. Distribution      | `distribution.md` | All five `distribution.md` required sections |
| 6. Constraints       | `constraints.md`  | All five `constraints.md` required sections  |

### Consolidation engine — **hybrid parser + LLM polish (OQ-B-02)**

The consolidator MUST use a two-pass design:

| Pass | Engine               | Handles                                                                                                                                                                        |
| ---- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Deterministic parser | Markdown AST walk over INTERVIEW.md. Extracts structured fields (checkbox state, tables, bullet/numbered lists, code blocks) **byte-verbatim**. No LLM involvement             |
| 2    | LLM polish           | Only the **free-text prose answers** are passed to Claude for voice normalization. LLM is sandboxed to the prose region — never sees or modifies structured fields from pass 1 |

This makes idempotency provable for ≥80% of the content (structured fields) and preserves PR-01 (no invented facts) even when polish quality varies.

### Transformation rules

| Source content type                                  | Pass | Transformation                                                                                                         |
| ---------------------------------------------------- | ---- | ---------------------------------------------------------------------------------------------------------------------- |
| Checkbox state                                       | 1    | Preserved verbatim                                                                                                     |
| Tables                                               | 1    | Preserved verbatim                                                                                                     |
| Bullet / numbered lists                              | 1    | Preserved verbatim                                                                                                     |
| Code blocks                                          | 1    | Preserved verbatim                                                                                                     |
| Free-text prose                                      | 2    | Polished by Claude to match target file's voice; condenses to detail-level cap (PR-05) while preserving meaning        |
| `[skipped: <reason>]` markers                        | 1    | Replaced in the output file with `<!-- TODO: skipped during bootstrap — <reason> -->` callout                          |
| Unanswered (`> _Your answer here..._` still present) | 1    | Same as `[skipped]` with reason "unanswered"                                                                           |
| **User-added `##` sections** in target files         | —    | **Preserved verbatim across `--consolidate` re-runs** (per OQ-M-02 extensibility — see "Hand-edit preservation" below) |

### Polishing constraints (LLM pass)

When polishing free-text prose, Claude MUST:

| Rule  | What it means                                                                                                                                                                                                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PR-01 | Preserve the user's facts verbatim — no inferred additions or removals                                                                                                                                                          |
| PR-02 | Normalize voice (capitalization, punctuation, sentence flow) without changing meaning                                                                                                                                           |
| PR-03 | Do not infer answers for skipped or unanswered questions                                                                                                                                                                        |
| PR-04 | Preserve language (es input → es output — applies even though v1 ships English-only; LLM polish must not "correct" Spanish user input into English)                                                                             |
| PR-05 | **Enforce detail-level cap by condensation (OQ-B-05)** — if user's answer exceeds the cap ("one sentence" but they wrote five), condense during polish while preserving meaning. Do not silently truncate; do not warn-and-keep |

### Hand-edit preservation rule (OQ-M-02 + OQ-B-04)

The target files (`product.md`, `audience.md`, etc.) may contain two kinds of user-added content:

| Kind                                                                                   | Treatment on `--consolidate` re-run                                                                                                                                                                         |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hand edits **inside required sections** (e.g. tweaked wording in `## Personas`)        | **Overwritten silently** per OQ-B-04. Git is the safety net. Documented in skill body so users know `--consolidate` regenerates required sections                                                           |
| User-added **extra `##` sections** beyond the required set (per OQ-M-02 extensibility) | **Preserved verbatim**. The consolidator detects existing target files, extracts any sections whose headers aren't in the required-section list, and re-attaches them at the bottom of the regenerated file |

### Freshness diff — **not emitted (OQ-B-03)**

`--consolidate` does **not** produce a diff of what changed since the last run. Rationale: git already does this. Adding a custom diff is v1 feature creep.

### Idempotency

| Property                            | Spec                                                                                                                                                   |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Same input → same output            | INTERVIEW.md unchanged + no extra `##` sections in target files → re-running `--consolidate` produces byte-identical target files                      |
| Pass-1 idempotency                  | Deterministic parser output is always byte-identical for the same input (no LLM nondeterminism)                                                        |
| Pass-2 idempotency tolerance        | LLM polish is **not** byte-identical run-to-run, but MUST be semantically equivalent — measured by manual review on the empleo.digital acceptance test |
| Hand edits inside required sections | **Overwritten** (per OQ-B-04, silent — see hand-edit preservation table)                                                                               |
| Hand-added extra sections           | **Preserved** verbatim (per OQ-M-02 extensibility)                                                                                                     |
| Documented                          | Bootstrap skill body documents INTERVIEW.md as the source of truth for required sections; target files own extra sections                              |

## Behavior specifications

### Initial scaffold (English default)

**Given** project root has no `agent_docs/marketing/` **When** user runs `/marketing-board:bootstrap` **Then** `agent_docs/marketing/INTERVIEW.md` is created **And** the file contains the 6 sections in order, each with its "ask Claude to interview me" hint **And** every question follows the locked render format (Why + Example + Detail level + answer container + skip marker) **And** language is English

### `--lang <code>` rejected (deferred to v1.1, OQ-B-06)

**Given** project root has no `agent_docs/marketing/` **When** user runs `/marketing-board:bootstrap --lang es` (or any other `--lang` value) **Then** the skill aborts with: `"--lang is not available in v1. Bootstrap ships English-only; non-English support arrives in v1.1."` **And** no files are created

### Abort on existing knowledge base

**Given** `agent_docs/marketing/INTERVIEW.md` exists **When** user runs `/marketing-board:bootstrap` (no flag) **Then** skill aborts with the exact abort message **And** no files are modified

### `--reset` regenerates INTERVIEW.md only

**Given** `agent_docs/marketing/INTERVIEW.md` and the six target files all exist **When** user runs `/marketing-board:bootstrap --reset` **Then** `INTERVIEW.md` is overwritten with a fresh scaffold **And** `product.md`, `audience.md`, `competition.md`, `business.md`, `distribution.md`, `constraints.md` are NOT touched **And** the user can re-fill INTERVIEW.md and `--consolidate` later to regenerate target files

### Consolidate produces all six files

**Given** `INTERVIEW.md` is filled in (every question answered, none skipped) **When** user runs `/marketing-board:bootstrap --consolidate` **Then** `product.md`, `audience.md`, `competition.md`, `business.md`, `distribution.md`, `constraints.md` are all written **And** each contains all its required `##` sections (per TDD 00) **And** structured fields (checkbox state, tables, lists) appear verbatim **And** no `<!-- TODO: -->` callouts appear in any file

### Consolidate writes business.md frontmatter (per TDD 03 OQ-06 refined)

**Given** INTERVIEW.md Q4.3 (Landing page URL) is answered with `https://empleo.digital` **When** user runs `/marketing-board:bootstrap --consolidate` **Then** `business.md` opens with a YAML frontmatter block: `---\nlp_url: https://empleo.digital\n---` followed by a blank line then `# Business` **And** `## Funnel state` body contains the prose answer from Q4.4 (signup flow + conversion rates) — without restating the LP URL

**Given** INTERVIEW.md Q4.3 is left blank (no LP URL yet) **When** user runs `/marketing-board:bootstrap --consolidate` **Then** `business.md` has no frontmatter block (or has frontmatter with `lp_url` omitted) — never an empty `lp_url:` key **And** consumers (e.g., the email-sequence skill) fall back to their placeholder behavior

### Consolidate with skipped questions

**Given** INTERVIEW.md has Q3.4 marked `[skipped: no real competitors yet]` **When** user runs `/marketing-board:bootstrap --consolidate` **Then** `competition.md` is still produced **And** the section that would have been populated by Q3.4 contains `<!-- TODO: skipped during bootstrap — no real competitors yet -->`

### Consolidate idempotency

**Given** INTERVIEW.md exists, filled **When** user runs `/marketing-board:bootstrap --consolidate` twice **Then** target files produced by the second run are byte-identical to the first

### Consolidate before scaffold

**Given** `agent_docs/marketing/INTERVIEW.md` does not exist **When** user runs `/marketing-board:bootstrap --consolidate` **Then** skill aborts with the "INTERVIEW.md not found" message **And** no files are written

## Acceptance criteria

### Skill structure

- [x] `plugins/marketing-board/skills/bootstrap/SKILL.md` exists
- [x] Frontmatter has `name`, `description`, `disable-model-invocation: true`
- [x] `templates/interview.md` exists (single English source per OQ-M-03 + OQ-B-06)
- [x] Template has all 6 sections with the 36 questions covering the locked Q-ID intents

### Scaffold behavior

- [x] Fresh scaffold produces `INTERVIEW.md` with 6 sections and 36 questions
- [x] Each section has the "ask Claude to interview me" hint
- [x] Every question matches the locked render format and corresponds to a Q-ID in the intent tables
- [x] Each question has at least one concrete worked example (not Lorem ipsum)
- [x] Multiple-choice questions use `- [ ]` checkboxes
- [x] Skip markers are present and explicit

### Flag behavior

- [ ] Re-invocation without flag on existing `agent_docs/marketing/` aborts with the exact message
- [ ] `--reset` regenerates INTERVIEW.md but does not touch consolidated target files
- [ ] `--consolidate` without existing INTERVIEW.md aborts with the exact message
- [ ] `--lang <any>` aborts with the v1.1-deferral message (per OQ-B-06)
- [ ] `--reset --consolidate` aborts with the contradictory-intent message

### Consolidation

- [x] All 6 target files written under `agent_docs/marketing/`
- [x] Each target file contains all required `##` sections (per TDD 00)
- [ ] Hybrid engine (OQ-B-02): structured fields (checkboxes, tables, lists, code blocks) go through pass-1 parser verbatim; only free-text prose goes through pass-2 LLM polish
- [ ] Free-text prose polished without inventing facts (PR-01)
- [ ] PR-05 detail-level cap is enforced by condensation, not silent truncation
- [ ] `[skipped]` markers produce `<!-- TODO: -->` callouts in output files
- [ ] Re-running `--consolidate` produces byte-identical pass-1 output (structured fields); pass-2 prose is semantically equivalent
- [ ] **User-added `##` sections in target files are preserved across `--consolidate` re-runs** (OQ-M-02 extensibility)
- [ ] Hand edits *inside* required sections are overwritten silently on re-run (OQ-B-04)
- [ ] No freshness diff emitted (OQ-B-03)

### Empleo.digital acceptance

- [x] Bootstrap runs successfully in `~/proyectos/empleo/`
- [x] User can fill INTERVIEW.md in English (v1 ships English-only per OQ-B-06)
- [x] `--consolidate` produces all six knowledge-base files
- [x] Files are accepted by `/marketing-board` (no hard-fail per TDD 01)

## Testing requirements

| Test                                           | Approach                                                                                                                                                                    |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scaffold lint**                              | Run scaffold; assert INTERVIEW.md has all 6 `## N. <Topic>` headers and 36 questions following the locked format                                                            |
| **Q-ID intent coverage**                       | For each Q-ID in the intent tables, assert the template has a matching `### QN.M:` block and the question covers the documented intent                                      |
| **Abort message exactness**                    | For each abort scenario (incl. `--lang <any>` rejection), assert the displayed message matches the spec verbatim                                                            |
| **Reset preservation**                         | Pre-create dummy target files; run `--reset`; assert dummy files untouched, INTERVIEW.md fresh                                                                              |
| **Consolidate pass-1 idempotency**             | Run `--consolidate` twice on same INTERVIEW.md; diff pass-1 output (structured fields); assert empty                                                                        |
| **Consolidate pass-2 equivalence**             | Manual: run twice; assert pass-2 prose is semantically equivalent (acceptable variance in wording)                                                                          |
| **Hybrid engine boundary**                     | Inspect implementation: assert structured fields never pass through the LLM and free-text prose never bypasses it                                                           |
| **Skipped → TODO callout**                     | Prepare INTERVIEW.md with `[skipped: test]`; run `--consolidate`; grep target file for `<!-- TODO:` callout                                                                 |
| **Verbatim preservation**                      | Prepare INTERVIEW.md with specific checkbox state + table; run `--consolidate`; assert state/table appear unchanged                                                         |
| **No-invention rule (PR-01)**                  | Manual: review consolidated `audience.md` against the INTERVIEW.md source; assert no facts appear that weren't in source                                                    |
| **Detail-level cap (PR-05)**                   | Pre-load INTERVIEW.md with a 5-sentence answer where "one sentence" was specified; run `--consolidate`; assert output is condensed to one sentence while preserving meaning |
| **Extra-section preservation**                 | Hand-edit `audience.md` to add a `## Notes` section; re-run `--consolidate`; assert the `## Notes` section remains intact and the required sections are regenerated         |
| **Hand-edit overwrite (in required sections)** | Hand-edit `## Personas` in `audience.md`; re-run `--consolidate`; assert hand edit is overwritten (no warning issued — OQ-B-04)                                             |
| **No freshness diff (OQ-B-03)**                | Inspect `--consolidate` output: assert no diff is printed to stdout                                                                                                         |
| **--lang rejection**                           | Run `/marketing-board:bootstrap --lang es`; assert the v1.1-deferral message is displayed and no files are created                                                          |
| **Empleo end-to-end**                          | Manual: bootstrap → fill → consolidate → run `/marketing-board` on empleo.digital; assert no hard-fail                                                                      |

## Authorization

N/A — see TDD 00.

## Related Documents

- **Master TDD**: [`00-marketing-board-master.md`](00-marketing-board-master.md)
- **Sibling TDD** (Deliberation): [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md)
- **Parent PRD**: [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md)
- **Pattern**: [`plugins/bootstrap-vault/`](../../../plugins/bootstrap-vault/) (sibling marketplace plugin with scaffold + consolidate flow)

## Open Questions

*All questions resolved and integrated as of 2026-05-20.*

### Resolution log

| ID      | Question (short)             | Resolution                                                                                                                                                                                                         | Integrated in                                                                                                                                     |
| ------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| OQ-B-01 | Locking question text in TDD | **Option 3 — lock intent (applied per Claude's recommendation; OQ checkbox was unmarked)**. Each question is locked to an intent + answer type in the per-section intent tables; template owns the literal wording | New "Question intent tables" subsection (~50 lines, 6 tables, 36 rows)                                                                            |
| OQ-B-02 | Consolidation engine         | **Hybrid** — deterministic parser handles structured fields (pass 1); LLM handles only free-text prose (pass 2)                                                                                                    | New "Consolidation engine" subsection; transformation rules now show pass column                                                                  |
| OQ-B-03 | Freshness diff               | **No diff** — git is the safety net                                                                                                                                                                                | New explicit "Freshness diff — not emitted" subsection; testing requirement                                                                       |
| OQ-B-04 | Hand-edit warning            | **Silent overwrite** — documented in skill body, git is the safety net                                                                                                                                             | New "Hand-edit preservation rule" subsection; idempotency table; acceptance criterion                                                             |
| OQ-B-05 | Detail-level enforcement     | **Enforce by condensation during LLM polish** — preserve meaning, cap output to declared detail level. Don't silently truncate; don't warn-and-keep                                                                | PR-05 updated; new test "Detail-level cap (PR-05)"                                                                                                |
| OQ-B-06 | Localization scope           | **`en` only in v1; `--lang` flag + LLM translation deferred to v1.1.** Architecture per OQ-M-03 is preserved (single English source + runtime translation); v1 just doesn't expose `--lang` yet                    | Scope, FR-16 (TDD 01), flag contracts, abort messages, behavior specs, testing — multiple sections updated; cross-doc impact on TDD 00 and TDD 01 |

### Cross-document propagation (applied)

- **From master TDD's OQ-M-02** (extensibility): consolidator now preserves user-added `##` sections in target files verbatim across re-runs (new "Hand-edit preservation rule" subsection + acceptance criterion + test)
- **From master TDD's OQ-M-03 + this TDD's OQ-B-06**: single English `templates/interview.md`; no per-language template files; `--lang` rejected in v1
- **To master TDD** (queued — see consolidation summary): "Locale handling" subsection needs note that v1 ships English-only; `--lang` flag arrives in v1.1 (architecture from OQ-M-03 is still correct)
- **To TDD 01** (queued — see consolidation summary): hard-fail message in FR-15 currently reads `"...bootstrap (--lang es for Spanish)."` — needs `--lang es` reference removed since flag isn't in v1
