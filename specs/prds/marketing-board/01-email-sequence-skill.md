# PRD — Email Sequence Skill

| Field        | Value                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Type         | Feature PRD (child of `marketing-board` master)                                   |
| Status       | v1.2 — OQs consolidated (11/11 resolved); ready for TDD                           |
| Owner        | Adrián Ribao                                                                      |
| Created      | 2026-05-22                                                                        |
| Parent PRD   | [`00-marketing-board-master.md`](00-marketing-board-master.md)                    |
| Sibling PRDs | (future) `02-video-script-skill.md`, `03-lp-audit-skill.md`                       |
| First test   | empleo.digital — the saved memo at `marketing-plans/empleo-digital-2026-05-22.md` |
| Packaging    | Skill inside the existing `marketing-board` plugin                                |
| Invocation   | `/marketing-board:email-sequence`                                                 |

______________________________________________________________________

## Problem & background

The marketing-board deliberation produces a Marketing Plan Memo whose Funnel Engineer report includes an `### Email sequences (outlines)` block: each sequence has a name (Welcome / Activation / Trial-to-paid / Win-back / etc.), a trigger, a target metric, an email count, and one line describing each email. That outline is genuinely useful for *strategy* — it tells the founder which sequences matter and what each email is for — but it stops short of anything the founder can paste into their ESP.

Today, the founder takes the outline, opens a doc, and writes each email manually: pulling adjectives from `agent_docs/marketing/product.md`'s brand voice, lifting buyer vocabulary from `audience.md`, drafting 2-3 subject-line variants for A/B testing, writing the body, choosing a CTA. For a 3-sequence plan with 4 emails per sequence and 2-3 variants of subject and CTA, that is ~12 emails × ~5 micro-decisions each = a half-day of careful work that pulls the founder away from talking to customers.

The skill closes this gap. It consumes the saved memo, drafts the full text of every email in every sequence (subject variants, full body, CTA variants), saves to `email-sequences/<sequence>-<date>.md`, and lets the founder iterate on the saved files via the same `--consolidate` mechanism marketing-board already uses — tick variant checkboxes, edit prose, run consolidate, drafts tighten in place.

The master PRD reserves this filename (`01-email-sequence-skill.md`) and lists the skill in "Out of scope (v1) — future child PRDs." This document promotes the placeholder to a real spec.

## Goals

1. **Close the outline-to-draft gap.** Single invocation against a memo produces ESP-paste-ready drafts for every sequence in the memo — including brand-calibrated visual prompts and email-safe HTML embed templates for any email referencing a video, hero image, or inline graphic. No more half-day rewrites from scratch.
2. **Preserve voice and vocabulary discipline.** Drafts pull adjectives and example sentences from `product.md`'s `## Brand voice & tone`; subjects and bodies use buyer terms from `audience.md`'s `## Vocabulary`. The drafts read as the brand, not as a generic SaaS template.
3. **A/B variant choice as a first-class deliverable.** Every email ships with 2-3 subject variants and 1-2 CTA variants as `- [ ]` checkboxes the founder ticks in-file. `--consolidate` bakes the picks in.
4. **Iterate via the same in-file mechanism the founder already knows.** No new mental model — `--consolidate` works exactly like marketing-board's: edit the file, run it, drafts get tightened in place.

## Non-goals (v1)

- **Not** generating ESP-specific output formats (Mailchimp MJML, ConvertKit / Customer.io syntax, AMP for Email). v1 is plain Markdown; the founder pastes into their ESP.
- **Not** sending emails or wiring ESP automations.
- **Not** generating personalization-token syntax (`{first_name}`, merge fields). Drafts use plain language with `<placeholder>` hints; ESP-specific tokens get substituted at paste time.
- **Not** proposing *new* sequences the memo's Funnel Engineer didn't already outline. The memo is the authoritative list of what to draft.
- **Not** drafting from a free-form brief without a memo. The memo is required.
- **Not** re-running the deliberation. If the founder wants different sequences, they re-run `/marketing-board <brief>`.
- **Not** the v2 sibling production skills (video-script, lp-audit) — those are separate future child PRDs.

## Target user

Same as marketing-board: a **SaaS founder with a saved Marketing Plan Memo in hand** who is post-deliberation and ready to ship the launch assets. The Funnel Engineer's outline tells them they need (for example) a 5-email Welcome sequence + a 3-email Activation sequence + a 4-email Win-back sequence. They need drafts now, not next quarter. The acceptance test is the same founder who built marketing-board v0.2.x's first memo on empleo.digital.

Secondary use: the same founder *iterating* on a sequence after first draft — picking subject/CTA variants in the saved file, rewriting a line of body prose where the brand voice missed, then running `--consolidate` to tighten everything in place.

## User workflows

### Workflow 1 — First-draft every sequence in a memo

1. The founder has a saved memo at `marketing-plans/<product>-<YYYY-MM-DD>.md` (produced by marketing-board v0.2.x always-save). The memo's Funnel Engineer report lists 2-4 sequences as outlines.
2. The founder runs `/marketing-board:email-sequence`. The skill auto-locates the most recently written memo file; reads it; reads `agent_docs/marketing/{product,audience}.md`; hard-fails if any required file or knowledge-base section is missing.
3. The skill drafts every email in every sequence — subject + 2-3 subject variants, preheader + 2 preheader variants, full body (length per sequence type — see FR-08), CTA + 1-2 CTA variants, all as `- [ ]` checkboxes the founder picks — and saves one file per sequence to `email-sequences/<sequence-slug>-<YYYY-MM-DD>.md`. Creates `email-sequences/` if absent. Suffixes with `-<HHMMSS>` on same-day collision so an annotated file is never overwritten.
4. The skill reports back: paths created + one-line reminder of how to pick variants and run `--consolidate`.

**Estimated time:** 1-3 minutes generation (one Claude turn synthesizing all sequences); the founder's review-and-pick step depends on sequence count.

### Workflow 2 — Single-sequence drafting

1. The founder wants only the Welcome sequence drafted right now (others can wait).
2. `/marketing-board:email-sequence welcome` (positional) or `--sequence welcome`.
3. The skill drafts only that sequence, writes one file (`email-sequences/welcome-<YYYY-MM-DD>.md`).
4. Iteration as in Workflow 1.

### Workflow 3 — Iteration on a drafted sequence

1. The founder opens `email-sequences/welcome-2026-05-22.md`. For each email: ticks one `- [x]` subject variant, ticks one CTA variant, rewrites a sentence of body prose by hand.
2. The founder runs `/marketing-board:email-sequence --consolidate` (most recent sequence file) or `--consolidate @<path>` (explicit).
3. The skill reads the file, keeps each email's ticked variants, removes the unticked options, preserves any prose edits verbatim, writes the consolidated version back to the same file in place.
4. Idempotent — running `--consolidate` again on the same file + same picks produces a byte-identical file.

### Workflow 4 — Re-draft after the memo changes

1. The founder re-ran `/marketing-board <new brief>` and got a new memo with different sequence outlines.
2. `/marketing-board:email-sequence` (no args) drafts against the new most-recent memo. Same-day collision → `-<HHMMSS>` suffix; previous drafts are not clobbered.

## Inputs (what the skill reads)

| Source                                                        | What's extracted                                                                                                       |
| ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Memo file (default: most recent in `marketing-plans/`)        | Funnel Engineer's `### Email sequences (outlines)` — sequence names, triggers, target metrics, counts, one-liners      |
| `agent_docs/marketing/product.md` `## Brand voice & tone`     | Adjectives + the example sentence the founder gave at bootstrap — voice calibration for every email                    |
| `agent_docs/marketing/product.md` `## Value propositions`     | The three-to-five proposition bullets — material for body content                                                      |
| `agent_docs/marketing/audience.md` `## Personas`              | Persona name, JTBD, pains — for "talk to the right human"                                                              |
| `agent_docs/marketing/audience.md` `## Vocabulary`            | Buyer terms to use; insider terms to avoid                                                                             |
| `agent_docs/marketing/audience.md` `## Triggers to buy`       | Trigger events worth invoking in subject lines and openers                                                             |
| `agent_docs/marketing/business.md` `## Funnel state`          | LP URL — for CTA placeholder substitution when available                                                               |
| `agent_docs/marketing/constraints.md` `## Legal / compliance` | Email-relevant compliance constraints (GDPR, CAN-SPAM, etc.) — drives conditional inline compliance reminder per FR-24 |

The skill does NOT read `competition.md` or `distribution.md` — those don't shape email copy in a way that justifies the token cost. `constraints.md` is read conditionally per FR-24.

## Output (what the skill writes)

| Aspect                           | Spec                                                                                                                                                                                                                                                                                                                                                                                                                 |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Output directory                 | `email-sequences/` in the working directory (sibling to `marketing-plans/`). Created if absent.                                                                                                                                                                                                                                                                                                                      |
| Filename                         | `<sequence-slug>-<YYYY-MM-DD>.md` — one file per sequence. Slug is the sequence name (from the memo's outline) kebab-cased.                                                                                                                                                                                                                                                                                          |
| Same-day collision               | Append `-<HHMMSS>` suffix. Never overwrite an existing memo file.                                                                                                                                                                                                                                                                                                                                                    |
| Per-file preamble                | Sequence name, trigger, target metric (from the memo) + voice notes (adjective list from `product.md`) + the intended persona name (from `audience.md`). When `product.md` is newer than the source memo, a freshness warning is added (per FR-22)                                                                                                                                                                   |
| Per-email block shape            | `## <NN> — <email name>`, `**Trigger:** ...`, `**Target metric:** ...`, `**Subject (pick one):**` + 2-3 `- [ ]` options, `**Preheader (pick one):**` + 2 `- [ ]` options, `**Body:**` prose (length per sequence type — see FR-08), optional `**Visual (thumbnail / hero / inline):**` block when the LLM judges a visual would strengthen the email (per FR-18, FR-19), `**CTA (pick one):**` + 1-2 `- [ ]` options |
| Optional sequence-level OQ block | When the LLM identifies sequence-level execution-blocking decisions, a `## Open Questions` block in marketing-board's locked OQ render shape is appended at the bottom of the file (per FR-25). Empty / absent when no blockers exist                                                                                                                                                                                |
| File language                    | Follows the memo's language. Spanish memo → Spanish drafts.                                                                                                                                                                                                                                                                                                                                                          |
| Format                           | Plain Markdown. No ESP-specific syntax in v1.                                                                                                                                                                                                                                                                                                                                                                        |

## Requirements

### Functional (FR)

| ID    | Requirement                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Priority |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| FR-01 | Skill lives at `plugins/marketing-board/skills/email-sequence/SKILL.md`. Frontmatter: `name: email-sequence`, `disable-model-invocation: true`. Invoked as `/marketing-board:email-sequence`                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | P0       |
| FR-02 | Skill auto-locates the most recently modified `.md` file in `marketing-plans/` when no path argument is supplied. `@<path>` arg overrides                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | P0       |
| FR-03 | Skill hard-fails with exact message `"No memo found. Run /marketing-board <brief> first."` if no memo file is found and no `@<path>` was supplied                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | P0       |
| FR-04 | Skill hard-fails with exact message `"Memo has no parseable '### Email sequences (outlines)' section. The Funnel Engineer seat must have produced an outline. Re-run /marketing-board to regenerate."` if the memo lacks the section                                                                                                                                                                                                                                                                                                                                                                                                                                      | P0       |
| FR-05 | Skill hard-fails if `agent_docs/marketing/product.md` or `agent_docs/marketing/audience.md` is missing or lacks any required section the skill reads. Message mirrors marketing-board's: `"Bootstrap your product context first: /marketing-board:bootstrap"`                                                                                                                                                                                                                                                                                                                                                                                                             | P0       |
| FR-06 | Default invocation drafts every sequence in the memo's outline (no skipped sequences)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | P0       |
| FR-07 | `--sequence <name>` (or positional `<name>`) scopes drafting to one sequence by name (case-insensitive match against the memo's sequence-name list)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | P1       |
| FR-08 | Every drafted email has: `## NN — <name>` header; `**Trigger:**`; `**Target metric:**`; `**Subject (pick one):**` block with 2-3 `- [ ]` variants; `**Preheader (pick one):**` block with 2 `- [ ]` variants; `**Body:**` block with body prose at the sequence-type-driven length default — Welcome and Win-back 100-180 words, Activation 150-250, Trial-to-paid 200-350; unrecognized sequence names fall back to 150-250; `**CTA (pick one):**` block with 1-2 `- [ ]` variants. When the LLM judges a visual would strengthen the message, also a `**Visual:**` block between Body and CTA (per FR-18, FR-19)                                                        | P0       |
| FR-09 | Body prose calibrates voice from `product.md`'s `## Brand voice & tone` (adjectives + example sentence are non-negotiable inputs to the synthesis turn). Vocabulary uses buyer terms from `audience.md`'s `## Vocabulary` two-column table                                                                                                                                                                                                                                                                                                                                                                                                                                | P0       |
| FR-10 | Output files written to `email-sequences/<sequence-slug>-<YYYY-MM-DD>.md`. Directory created if absent. Same-day collision → `-<HHMMSS>` suffix. Never overwrite an existing memo file                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-11 | Drafts follow the memo's language, detected implicitly by the LLM during the synthesis turn (the prompt passes memo content; Claude naturally writes in matching language — no explicit detection step). `--lang <code>` flag deferred to v1.1 as an explicit override                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-12 | `--consolidate [<sequence>] [@<path>]` reads the saved sequence file, applies the user's variant picks (`- [x]`) by keeping the chosen options and removing the others, preserves the user's prose edits verbatim, and writes the consolidated memo back to the same file in place. Edge cases: when an email has no variant ticked (or all variants ticked), the skill keeps all variants for that email and adds it to a "still needs decisions" list in the run summary — lossless and visible, never silently picks                                                                                                                                                   | P0       |
| FR-13 | `--consolidate` is idempotent — re-running on the same file + same in-file picks produces a byte-identical file                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | P0       |
| FR-14 | Skill tool allowlist limited to `Read, Grep, Glob, Write, Edit`. No Bash, no WebSearch                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-15 | After successful drafting, the skill writes a short summary to the conversation: the paths created and a one-line reminder of how to iterate (tick variant checkboxes, edit prose, run `--consolidate`)                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | P1       |
| FR-16 | When the memo language differs from the knowledge-base files' language, the skill follows the memo. (e.g., English KB + Spanish memo → Spanish drafts. The knowledge base provides facts; the memo decides voice.)                                                                                                                                                                                                                                                                                                                                                                                                                                                        | P1       |
| FR-17 | A `## Drafted from` line at the top of every generated file records the source memo path and the date, so the founder can trace any draft back to its memo                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | P1       |
| FR-18 | The synthesis turn judges per email whether a visual (video thumbnail, hero image, inline graphic) would meaningfully strengthen the message. When yes, the per-email block includes a `**Visual:**` section with three artifacts: (a) a tool-agnostic image-generation prompt suitable for Gemini Nano Banana Pro / Imagen / Midjourney / Ideogram; (b) accessible alt-text; (c) an email-safe HTML embed template with `<VIDEO_URL>` and `<THUMBNAIL_URL>` (or `<IMAGE_URL>`) placeholders the founder substitutes at paste time. Video-thumbnail prompts explicitly include "centered play-button overlay". No external keyword matching — the model decides per email | P0       |
| FR-19 | Visual prompts are brand-calibrated: palette / mood references derive from `product.md`'s `## Brand voice & tone`; the scene references the sequence's intended persona (name + day-in-the-life) from `audience.md`'s `## Personas`. Prompts are tool-agnostic — no Gemini-specific syntax in v1                                                                                                                                                                                                                                                                                                                                                                          | P0       |
| FR-20 | The skill does NOT call any image-generation API in v1 (no Gemini / Imagen / Midjourney API calls). It emits prompts only. Files are written as Markdown text; no binary image outputs                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |

| FR-21 | When the Funnel Engineer outline names a sequence and email count but lacks per-email descriptions, the skill invents the per-email arc ONLY IF the outline supplies sequence name + trigger + target metric (the three signals deemed sufficient). When any of those three is missing, the skill hard-fails with: `"Sequence '<name>' lacks the minimum context (trigger or target metric) needed to infer per-email content. Edit the memo to add this context, then re-run."`. When inference does happen, every inferred per-email block is marked with a `<!-- INFERRED: outline was thin; review carefully -->` callout | P0 |
| FR-22 | Before drafting, the skill compares the modification time of `agent_docs/marketing/product.md` to the source memo's modification time. When `product.md` is newer, the skill emits a one-line warning in the run summary: `"Brand voice has been updated since this memo was generated; drafts use the current voice. Re-run /marketing-board for a consistent memo."`. Drafts always use the current KB, never the memo's implicit snapshot | P1 |
| FR-23 | When body or CTA references the product's landing page, the skill substitutes the LP URL inline if `agent_docs/marketing/business.md`'s `## Funnel state` contains it. For other URLs the skill doesn't know (demo video, social-proof page, etc.), it emits an obvious placeholder (`<https://YOUR-DEMO-URL>` form) the founder substitutes at paste time | P1 |
| FR-24 | The skill reads `agent_docs/marketing/constraints.md`'s `## Legal / compliance` section. When that section names GDPR, CAN-SPAM, or similar email-relevant compliance constraints, the skill injects an inline compliance reminder (unsubscribe-link expectation + sender-address reminder) into the appropriate email or a once-per-sequence `## Compliance check` block. When the section is "None" or names no email-relevant constraints, no compliance text is added — drafts stay clean | P1 |
| FR-25 | When the LLM identifies execution-blocking decisions for a sequence as a whole (e.g., "the trigger 'signup verified' isn't defined in `product.md` — which event in your funnel?"), the skill emits a `## Open Questions` block at the bottom of that sequence file in the same locked render shape as marketing-board's memo OQs (intro line + summary table + per-OQ detail blocks with checkboxed options). When no execution-blocking decisions exist, no OQ block is emitted | P1 |

### Non-functional (NFR)

| ID     | Requirement                                                                                                                                | Priority |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| NFR-01 | Skill fits in a single SKILL.md without supporting files (matches `:bootstrap`'s shape). If a template is needed it lives at `templates/`. | P1       |
| NFR-02 | All marketplace conventions per [CLAUDE.md](../../../CLAUDE.md) apply: version-bump on every change inside the plugin                      | P0       |
| NFR-03 | Voice fidelity is the load-bearing quality metric. Drafts that don't read as the brand are failure mode #1                                 | P0       |
| NFR-04 | One Claude turn drafts the whole memo by default. Cost-bounded by the memo size, not the sequence count                                    | P1       |
| NFR-05 | Skill is explicit-invocation only (`disable-model-invocation: true`). No auto-firing                                                       | P0       |

## Success metrics

| Metric                                                                                     | Baseline | Target                             | Measurement                                          |
| ------------------------------------------------------------------------------------------ | -------- | ---------------------------------- | ---------------------------------------------------- |
| Drafted sequences are usable with light editing (not "start over" rewrite)                 | N/A      | ≥3 of first 5 generations          | Founder qualitative review                           |
| Voice fidelity — drafts feel like the brand in `product.md`'s voice section                | N/A      | ≥4 of first 5 outputs              | Founder qualitative judgment                         |
| Buyer vocabulary used verbatim — terms from `audience.md` `## Vocabulary` appear in drafts | N/A      | ≥1 buyer term per email on average | Author greps `<sequence-file>.md` against vocabulary |
| Time from memo to ESP-paste-ready drafts                                                   | ~4 hours | \<45 minutes for a 3-sequence memo | Author self-report                                   |
| `--consolidate` idempotency (byte-identical re-runs)                                       | N/A      | 100%                               | Run twice, `diff` empty                              |
| First end-to-end test on empleo.digital produces a usable sequence file                    | N/A      | Pass on first run                  | Author runs against the saved empleo memo, reviews   |

## Scope

### In scope (v1)

- `/marketing-board:email-sequence` skill — drafts every sequence in the memo by default
- `--sequence <name>` (or positional) for single-sequence drafting
- `--consolidate [<sequence>] [@<path>]` for picking variants + tightening (mirrors marketing-board's `--consolidate`)
- File-based persistence under `email-sequences/<sequence-slug>-<YYYY-MM-DD>.md`
- Full body (150-250 words default) + 2-3 subject variants + 1-2 CTA variants per email
- Voice from `product.md`, vocabulary + persona + triggers from `audience.md`, LP URL from `business.md` (for CTA placeholder substitution when available)
- Language follows the memo
- Hard-fail on missing memo, missing outline section, or missing knowledge-base files
- Same-day collision suffix
- "Drafted from" provenance line at the top of every output file
- Visual prompts + email-safe HTML embed templates for any email referencing a video, hero image, or inline graphic (per FR-18 / FR-19). Prompts are tool-agnostic (Gemini Nano Banana Pro, Imagen, Midjourney, etc.) and brand-calibrated from `product.md` + `audience.md`. Video-thumbnail prompts explicitly include the play-button overlay
- An email-safe HTML embed template wrapping the thumbnail in a link to the video URL — both URLs as substitutable placeholders
- Preheader (2 `- [ ]` variants per email, between Subject and Body — per FR-08, OQ-08)
- Conditional compliance reminder when `constraints.md`'s `## Legal / compliance` names email-relevant constraints (GDPR, CAN-SPAM, etc.) — per FR-24, OQ-10
- Conditional sequence-level `## Open Questions` block when the LLM identifies execution-blockers for the sequence (per FR-25, OQ-05)
- KB freshness warning when `product.md` is newer than the source memo (per FR-22, OQ-07)
- LP URL substituted inline from `business.md`'s `## Funnel state` when available; obvious placeholders for other URLs (per FR-23, OQ-09)

### Out of scope (v1) — possible future enhancements

- ESP-specific export formats (Mailchimp MJML, ConvertKit / Customer.io, AMP for Email) — a sibling `esp-export` skill could read a consolidated sequence file and emit ESP-native syntax. Deferred.
- A `--refine "<intent>"` mode for prose-level regeneration under a constraint ("make subjects punchier under 6 words"). Deferred to v1.1 if `--consolidate` proves insufficient for editing needs in practice.
- Personalization-token wiring (`{first_name}`, merge tags). v1 leaves the founder to substitute at paste time.
- **Calling image-generation APIs from the skill** (no Gemini Nano Banana Pro / Imagen / Midjourney / Ideogram API calls in v1). v1 emits visual *prompts* + HTML embed templates only. v1.1 may add a `--generate-images` flag (gated on `GEMINI_API_KEY` or equivalent) once the prompt format is locked. Also out of scope v1: server-side compositing of play-button overlays onto existing video frames (the play-button overlay is an instruction inside the image-generation prompt, not a post-processing step)
- A drafted-sequence "audit" mode that scores an existing sequence against best practices. Different skill, deferred.
- Multi-persona variant generation — drafting alternate bodies per persona within the same sequence. Deferred; v1 picks one persona per sequence.

### Out of scope (forever)

- Sending emails
- Tracking opens / clicks / conversions
- Multi-step ESP automation wiring (segments, triggers, splits)
- A/B test execution and analysis — the skill *produces* variants but does not run the test

## Constraints & dependencies

### Constraints

- Must follow marketplace CLAUDE.md conventions (version-bump on every change inside the plugin).
- Must be additive to the marketing-board plugin — no breaking changes to the existing command, deliberation flow, agents, synthesizer, or bootstrap skill.
- Knowledge-base file shapes are fixed by TDD 00 master; this skill READS them but never defines them.
- Marketing Plan Memo shape is fixed by TDD 01; this skill PARSES the Funnel Engineer's `### Email sequences (outlines)` block. The coupling is acknowledged: if the memo template's email-sequences sub-shape changes, this skill must adapt.

### Dependencies

- **`marketing-board` plugin v0.2.x or later** — for the Marketing Plan Memo and the knowledge base.
- **A populated `agent_docs/marketing/`** — specifically `product.md` and `audience.md`. (`business.md` is optional but improves CTA quality.)
- **A saved Marketing Plan Memo** at `marketing-plans/<...>.md` (always-save behavior of marketing-board v0.2+).

No external services, no paid APIs.

## Architecture decisions (locked for v1)

These are scope decisions, not implementation detail:

1. **Skill, not a board.** Single-LLM-turn synthesis from an existing memo. No parallel-agent fan-out. Aligns with the "production skills" framing in the master PRD's out-of-scope section.
2. **File-per-sequence output.** One file per sequence under `email-sequences/`, mirroring marketing-board's always-save + per-deliberation-file convention. (Resolves the "where do drafts live" question with the obvious analogous answer.)
3. **Variant picking via in-file checkboxes.** Subjects and CTAs render as `- [ ]` lists; the founder ticks the chosen option; `--consolidate` keeps the ticked and removes the unticked. Reuses the founder's habit from marketing-board's OQ workflow — no new mental model.
4. **No `--refine` mode in v1.** `--consolidate` covers variant picks and verbatim-preserved prose edits. If founders find themselves wanting "regenerate with constraint X" in practice, we add `--refine` in v1.1.
5. **No ESP-specific export in v1.** Plain Markdown. ESP-native formats deferred to a sibling future skill (`esp-export`) rather than crammed into this skill.
6. **Skill inherits marketing-board's hard-fail discipline.** No memo or no knowledge base → hard-fail with a single actionable message. No partial-context fallback.
7. **Memo is the single source of truth for which sequences exist.** The skill never proposes new sequences not in the memo; it only drafts what the Funnel Engineer outlined. (Refinement of strategy belongs in marketing-board, not here.)
8. **Lives as a skill inside the marketing-board plugin**, not a separate plugin. Shares the knowledge base, the install footprint, and the user's mental model.
9. **One persona per sequence in v1; LLM judgment with explicit ask-back when uncertain.** Per sequence, the skill picks the intended persona via LLM judgment over the Funnel Engineer's trigger + target metric + the audience file's persona blocks. When the LLM's confidence is low (multiple personas equally apt for the trigger, or no clear signal), the skill pauses and explicitly asks the founder which persona to use for that sequence before drafting it. Multi-persona variant generation (drafting alternate bodies per persona within the same sequence) is deferred to v1.1.
10. **Visual content is prompts only, not generated files in v1.** The skill emits brand-calibrated, persona-aware image-generation prompts and email-safe HTML embed templates with URL placeholders. It does NOT call Gemini Nano Banana Pro / Imagen / Midjourney / Ideogram or any image-generation API; does NOT save image files; does NOT composite play-button overlays. The founder generates images in their preferred tool, hosts them, substitutes URLs into the embed HTML. v1.1 may add `--generate-images` once the v1 prompt format proves itself in real use.

## Related documents

- **Parent (master)**: [`00-marketing-board-master.md`](00-marketing-board-master.md) — reserves this filename in "Out of scope (v1) — future child PRDs."
- **TDD 01** (deliberation, defines the Funnel Engineer outline this skill parses): [`../../tdds/marketing-board/01-marketing-board-deliberation.md`](../../tdds/marketing-board/01-marketing-board-deliberation.md)
- **TDD 02** (bootstrap, defines the knowledge-base file shapes this skill reads): [`../../tdds/marketing-board/02-marketing-board-bootstrap.md`](../../tdds/marketing-board/02-marketing-board-bootstrap.md)
- **Sibling future PRDs** (not yet written):
    - `02-video-script-skill.md` — Gemini Omni / HeyGen Hyperframes script drafting from the memo
    - `03-lp-audit-skill.md` — landing-page conversion audit against the Funnel Engineer's plan
- **Sibling plugin pattern** (to mirror): [`plugins/marketing-board/skills/bootstrap/`](../../../plugins/marketing-board/skills/bootstrap/) — the existing skill inside marketing-board sets the precedent for skill structure, frontmatter, and `--consolidate` discipline.

## Open Questions

*All 11 questions resolved and integrated as of 2026-05-22.*

### Resolution log

| ID    | Question (short)                                    | Resolution                                                                                                                                                       | Integrated in                  |
| ----- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| OQ-01 | Persona-per-sequence selection rule                 | **LLM judgment with explicit ask-back when uncertain** (user-added 5th option). The skill judges per sequence; pauses and asks the founder if confidence is low. | AD-9 (rewritten)               |
| OQ-02 | Body length defaults                                | **Sequence-type-driven** — Welcome/Win-back 100-180, Activation 150-250, Trial-to-paid 200-350; fallback 150-250 for unrecognized sequence names.                | FR-08                          |
| OQ-03 | Outline lacks per-email descriptions                | **Hybrid: invent only if sequence name + trigger + target metric all present; otherwise hard-fail.** Inferred per-email content marked with `<!-- INFERRED -->`. | FR-21 (new)                    |
| OQ-04 | Memo language detection mechanism                   | **LLM detects implicitly during synthesis turn**; `--lang` flag deferred to v1.1.                                                                                | FR-11 (updated)                |
| OQ-05 | Top-level sequence-level OQ section in drafted file | **Conditional** — emit `## Open Questions` block only when the LLM identifies execution-blocking decisions.                                                      | FR-25 (new), Output table      |
| OQ-06 | `--consolidate` edge cases (no picks / all picks)   | **Keep all + warn in run summary** — lossless, surfaces per-email which still need decisions.                                                                    | FR-12 (updated)                |
| OQ-07 | KB freshness vs memo snapshot                       | **Current KB always + freshness warning** when `product.md` mtime > memo mtime.                                                                                  | FR-22 (new), Output table      |
| OQ-08 | Preheader text inclusion                            | **Always include `**Preheader (pick one):**` with 2 variants** between Subject and Body.                                                                         | FR-08, Output table            |
| OQ-09 | Link / URL placeholder convention                   | **Pull LP URL from `business.md`'s `## Funnel state` when present**; placeholder for other URLs.                                                                 | FR-23 (new)                    |
| OQ-10 | Compliance text in drafts                           | **Constraints-driven** — read `constraints.md`'s `## Legal / compliance`; inject reminder only when GDPR/CAN-SPAM/similar is named.                              | FR-24 (new), Inputs table      |
| OQ-11 | Which emails get a `**Visual:**` block              | **LLM judgment per email** — synthesis turn decides per-email whether a visual would strengthen the message.                                                     | FR-18 (updated)                |
