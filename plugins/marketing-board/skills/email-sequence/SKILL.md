---
name: email-sequence
description: Draft ESP-paste-ready email sequences from a saved Marketing Plan Memo. Generates subject/preheader/CTA variants as `- [ ]` checkboxes for in-file picking; `--consolidate` bakes the picks in. Explicit invocation only — `/marketing-board:email-sequence`.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Write, Edit
argument-hint: '[<sequence-name>] [@<memo-path>] [--consolidate]'
---

# Email Sequence

Drafts ESP-paste-ready email sequences from a saved Marketing Plan Memo. Each email gets subject / preheader / CTA variants rendered as `- [ ]` checkboxes. The founder ticks one per block in the saved file, then runs `--consolidate` to bake the picks in.

The skill never invents sequences — it drafts only what the memo's Funnel Engineer outline names. It never auto-fires; it runs only when the user types `/marketing-board:email-sequence`.

## When to use

Run after a `/marketing-board` deliberation has produced a saved memo at `marketing-plans/<product>-<YYYY-MM-DD>.md`. The memo's Funnel Engineer seat produces a `### Email sequences (outlines)` block; this skill turns each outline into full, paste-ready drafts.

Two modes:

- **Draft** (default) — read the memo, draft every sequence (or one named sequence), write a file per sequence.
- **Consolidate** (`--consolidate`) — read a drafted file, bake in the variant picks the founder ticked, rewrite in place.

## Inputs

Parse `$ARGUMENTS` first, then route per the **Flag handling** table below.

**Memo resolution (draft mode):**

- If `@<memo-path>` is supplied, use that file.
- Otherwise `Glob` `marketing-plans/*.md` and pick the **latest by the date in the filename** (`<product>-<YYYY-MM-DD>.md`); if several share that date, the one with the highest numeric `-N` suffix. Memo filenames always carry their date, so recency is read from the name — this skill cannot observe file modification times (no Bash in `allowed-tools`).
- If neither resolves to a real file -> hard-fail `HF-NO-MEMO`.

**Knowledge-base files read** (all under `agent_docs/marketing/` in the working directory):

| File             | What is read                                         | Required?   | Use                                                                                                 |
| ---------------- | ---------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------- |
| `product.md`     | `## Brand voice & tone`, `## Value propositions`     | Required    | Voice calibration; body content material                                                            |
| `audience.md`    | `## Personas`, `## Vocabulary`, `## Triggers to buy` | Required    | Persona scene; buyer terms; subject-line trigger material                                           |
| `business.md`    | YAML frontmatter `lp_url` key                        | Optional    | Landing-page URL substituted inline (see **LP URL**); deterministic frontmatter parse, no inference |
| `constraints.md` | `## Legal / compliance`                              | Conditional | Read always; compliance block emitted only when GDPR / CAN-SPAM / similar is named                  |

Do **not** read `competition.md` or `distribution.md` — they are out of scope for drafting emails.

If `product.md` or `audience.md` is missing, or lacks any of its required `##` sections -> hard-fail `HF-NO-KB`.

**LP URL:** read `business.md`'s YAML frontmatter and take the `lp_url` value. Substitute it inline wherever a drafted body or CTA references the landing page. If the frontmatter is absent or has no `lp_url` key, use the placeholder `<https://YOUR-LP-URL>` instead. Never infer the URL from prose.

### Flag handling

| Invocation                                              | Mode        | Behavior                                                                                   |
| ------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------ |
| `/marketing-board:email-sequence`                       | Draft       | Draft ALL sequences in the latest-dated `marketing-plans/*.md` memo                        |
| `/marketing-board:email-sequence <name>`                | Draft       | Draft ONLY the named sequence (positional `--sequence`); case-insensitive exact name match |
| `/marketing-board:email-sequence --sequence <name>`     | Draft       | Explicit form of the positional. Same behavior                                             |
| `/marketing-board:email-sequence @<memo-path>`          | Draft       | Use the specified memo instead of "most recent". Combinable with `<name>`                  |
| `/marketing-board:email-sequence --consolidate`         | Consolidate | Operate on the latest-dated `email-sequences/*.md` file                                    |
| `/marketing-board:email-sequence --consolidate <name>`  | Consolidate | Target the named sequence's file (slug match)                                              |
| `/marketing-board:email-sequence --consolidate @<path>` | Consolidate | Target the specified sequence file                                                         |

**Conflict rule:** `<name>` and `@<path>` may be combined in draft mode, but NOT alongside `--consolidate` (in consolidate mode the `@<path>` points to a sequence file, so a `<name>` is redundant and ambiguous). If `--consolidate`, a bare `<name>`, and an `@<path>` are all present at once -> hard-fail `HF-CONFLICTING-ARGS`.

## Hard-fail conditions

No partial-context fallback. No silent degradation. On any condition below, print the message **verbatim** and stop — write no files.

| ID                    | Condition                                                                | Message (verbatim)                                                                                                                                           |
| --------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| HF-NO-MEMO            | No memo file resolvable (draft mode)                                     | `No memo found. Run /marketing-board <brief> first.`                                                                                                         |
| HF-NO-OUTLINE         | Memo lacks a parseable email-sequences section                           | `Memo has no parseable 'Email sequences (outlines)' section. The Funnel Engineer seat must have produced an outline. Re-run /marketing-board to regenerate.` |
| HF-NO-KB              | `product.md` or `audience.md` missing, or a required `##` section absent | `Bootstrap your product context first: /marketing-board:bootstrap`                                                                                           |
| HF-THIN-OUTLINE       | A sequence outline lacks sequence name + trigger + target metric         | `Sequence '<name>' lacks the minimum context (trigger or target metric) needed to infer per-email content. Edit the memo to add this context, then re-run.`  |
| HF-CONSOLIDATE-NOFILE | `--consolidate` but no `email-sequences/*.md` file found                 | `No email-sequence file found to consolidate. Run /marketing-board:email-sequence first.`                                                                    |
| HF-CONFLICTING-ARGS   | `<name>`, `@<path>`, and `--consolidate` combined illegally              | \`Conflicting arguments. Use one of: <name>, @<path>, or --consolidate \[<name>                                                                              |

## Drafting flow

Run these steps in order. Extraction and drafting happen in **this same turn** — one synthesis pass, no separate parser round-trip.

1. **Parse args** and determine mode + scope. If args conflict -> `HF-CONFLICTING-ARGS`.
2. **Resolve the memo** per **Inputs**. If none -> `HF-NO-MEMO`.
3. **Read the memo** and locate the `Email sequences (outlines)` heading at any level (level-2 `##` or level-3 `###`) — match the heading text case-insensitively, tolerating extra whitespace and punctuation variants. Memos emit it as a top-level `## Email sequences (outlines)` section; the Funnel Engineer's raw report uses level-3. If absent -> `HF-NO-OUTLINE`.
4. **Verify the knowledge base.** Read `product.md` and `audience.md` and confirm every required `##` section is present. If anything is missing -> `HF-NO-KB`.
5. **Read the optional / conditional KB.** Parse `business.md` frontmatter for `lp_url`. Read `constraints.md`'s `## Legal / compliance` section.
6. **Extract sequences** from the outline. For each: `{name, trigger, target_metric, email_count, per_email_outlines[]}`.
    - Source of truth is the memo only — never add a sequence the outline does not name.
    - **Invention gate:** if `per_email_outlines` is empty but name + trigger + target metric are all present, infer the per-email arc from those three signals and mark each inferred block with `<!-- INFERRED: outline was thin; review carefully -->`.
    - If a sequence lacks trigger OR target metric -> `HF-THIN-OUTLINE` naming that sequence.
7. **Scope.** If a `<name>` was supplied, keep only the matching sequence (case-insensitive exact match against extracted names).
8. **Judge persona** for each in-scope sequence (see **Persona ask-back**). Collect every sequence whose persona is ambiguous.
9. **Ask back if needed.** If any sequence is persona-ambiguous, emit ONE batch message and wait for the founder's reply before drafting.
10. **Draft** every in-scope sequence in one synthesis pass, following **Output file shape** and **Per-email block shape**. Write in the memo's language (detect implicitly).
11. **Write files.** One file per sequence at `email-sequences/<slug>-<YYYY-MM-DD>.md` (create `email-sequences/` if absent). Apply the same-day collision rule (see **Idempotency & safety**).
12. **Emit the run summary** per **Run summary** below.

> **Deferred to v1.1 — freshness warning (FR-22).** Warning the founder when `product.md` changed after the memo was generated requires comparing file modification times across two directories. This skill has no Bash and cannot observe mtimes (Glob exposes only mtime-*ordered* paths within one pattern, capped at 100, with no timestamps), and `product.md` carries no date in its name to compare. The check is deferred until the skill can read mtimes reliably.

### Body length by sequence type

Match each sequence name against this lookup (case-insensitive substring). For a non-English name, translate it internally to the English-keyword space first (e.g. `Bienvenida` -> `welcome`, `Activación` -> `activation`, `Reenganche` -> `re-engagement`, `Conversión` -> `trial-to-paid`), then match. The table stays English-canonical.

| Match keyword(s)                                                  | Body length (words) |
| ----------------------------------------------------------------- | ------------------- |
| `welcome`, `onboarding`                                           | 100-180             |
| `activation`                                                      | 150-250             |
| `trial-to-paid`, `trial to paid`, `t2p`, `paid conversion`        | 200-350             |
| `win-back`, `winback`, `re-engagement`, `reengagement`, `dormant` | 100-180             |
| `nurture`, `education`                                            | 200-300             |
| (anything else)                                                   | 150-250             |

### Run summary (draft mode)

After writing files, report to the founder:

- **Extracted sequences + counts** — e.g. `Extracted 3 sequences from memo: Welcome (4 emails), Activation (5), Win-back (3).`
- **Saved paths** — one bullet per file written.
- **Re-run guidance**, verbatim: `If a sequence in the memo is missing here, re-run with /marketing-board:email-sequence <name> @<memo-path>.`
- **Variant-pick reminder:** `Tick - [x] on one variant per Subject / Preheader / CTA block, then run /marketing-board:email-sequence --consolidate.`

## Persona ask-back

For each in-scope sequence, judge which persona from `audience.md`'s `## Personas` the sequence addresses. Use your own judgment — there is no numeric or lexical threshold. A sequence is **ambiguous** when more than one persona plausibly fits its trigger and target metric.

If one or more sequences are ambiguous, emit a SINGLE message at the top of the turn listing all of them, then wait for one reply, then draft everything in one pass:

```
Persona-ambiguous sequences detected — pick one for each:

- Activation — Carlos, Marta, or Luis?
- Trial-to-paid — Marta or Luis?

Reply with your picks (e.g. "Activation: Marta. Trial-to-paid: Luis."), and I'll draft everything.
```

Rules:

- No ambiguity -> skip the message entirely; draft directly.
- Ambiguity in only one sequence -> still use this format (one bullet).
- If the reply does not cover every asked sequence -> re-ask only the unanswered ones, once.
- One pause maximum per run — never ask sequence-by-sequence.

## Consolidate flow

Triggered by `--consolidate`. Bakes in the founder's variant picks and rewrites the file in place.

1. **Locate the file.**
    - With `@<path>` -> use that file.
    - With `<name>` -> filter `email-sequences/*.md` to filenames whose slug matches `<name>` (case-insensitive).
    - Otherwise -> consider all `email-sequences/*.md`.
    - Among candidates, pick the **latest by the date in the filename** (`<slug>-<YYYY-MM-DD>.md`); if several share that date, the highest numeric `-N` suffix. (Recency is read from the filename — the skill cannot observe mtimes.) If zero candidates -> `HF-CONSOLIDATE-NOFILE`.
2. **Read** the chosen file and parse each email's variant blocks (Subject / Preheader / CTA) and their `- [x]` / `- [ ]` state.
3. **Apply picks per block:**
    - Exactly one variant ticked -> collapse the block to a single line (see **Variant collapse** below); remove the unticked alternatives.
    - Zero ticked, or more than one ticked -> keep the block unchanged and add this email to the "still needs decisions" list.
4. **Preserve prose.** Any edit the founder made to body content, voice notes, or visual blocks is kept verbatim — only variant blocks (and resolved OQ blocks) change.
5. **Resolve sequence-level OQ block if present** (see **Sequence-level OQ consolidate** below).
6. **Rewrite in place** — same path, no new file.
7. **Emit the consolidate run summary:** the chosen file path plus, when picked without `@<path>`, the hint `To consolidate a different file, pass @<path>.`; the count of variant blocks collapsed; the count of OQs resolved (if any); and the "still needs decisions" list (emails with no pick or over-ticked, with a one-line hint for the over-ticked case).

### Variant collapse

| Before (`- [x]` on one variant)                                            | After                       |
| -------------------------------------------------------------------------- | --------------------------- |
| `**Subject (pick one):**` + `- [x] First pick` + `- [ ] Alt` + `- [ ] Alt` | `**Subject:** First pick`   |
| `**Preheader (pick one):**` + `- [x] First pick` + `- [ ] Alt`             | `**Preheader:** First pick` |
| `**CTA (pick one):**` + `- [x] First pick` + `- [ ] Alt`                   | `**CTA:** First pick`       |

Re-running `--consolidate` on an already-collapsed file is a byte-identical no-op: a collapsed block has no checkboxes left to pick, so nothing changes.

### Sequence-level OQ consolidate

If the file has a `## Open Questions` block (per **Output file shape**) and the founder ticked answers, mirror `/marketing-board`'s memo-consolidate behavior:

1. Fold each ticked answer into the relevant per-email block (e.g. update a `**Trigger:**` line when the OQ resolved which trigger to use).
2. Mark each resolved OQ's summary-table row `Resolved (YYYY-MM-DD) — <one-line answer>`.
3. Remove the resolved OQ's `### OQ-NN — <title>` detail block; keep its summary-table row.
4. When ALL OQs are resolved, replace the whole `## Open Questions` block with a one-line log: `*All N sequence-level decisions resolved on YYYY-MM-DD and integrated above.*`

Unresolved OQs (nothing ticked) stay visible with their full detail blocks.

## Output file shape

One file per sequence at `email-sequences/<slug>-<YYYY-MM-DD>.md`. Slug rules in **Idempotency & safety**.

Conditional blocks emit ONLY when their trigger holds — an untriggered block is absent entirely (never an empty heading).

```markdown
# <Sequence name> sequence — <product>

> Drafted from: marketing-plans/<product>-<YYYY-MM-DD>.md

**Trigger:** <from memo>
**Target metric:** <from memo>
**Persona:** <name from audience.md>
**Voice notes:** <adjective list from product.md>

## Compliance check
<!-- emitted only when constraints.md `## Legal / compliance` names GDPR / CAN-SPAM / similar -->

- <constraint-specific reminder, e.g. unsubscribe link + physical sender address required>

## 01 — <email name>

<per-email block — see Per-email block shape>

## 02 — <email name>

<per-email block>

## Open Questions
<!-- emitted only when an execution-blocking decision was surfaced for this sequence -->

*Sequence-level decisions surfaced during drafting — resolve in this file, then run `--consolidate`.*

| ID    | Question             | Status |
| ----- | -------------------- | ------ |
| OQ-01 | <one-line question>  | Open   |

### OQ-01 — <Title>

**Question:** <what must be decided>

**Why it matters:** <execution consequence>

**Possible answers:**

- [ ] <Option A>
- [ ] <Option B>

**Status:** Open — recommend <Option> because <reason>
```

## Per-email block shape

Every drafted email has these elements in this order. Subject offers 2-3 variants, Preheader 2, CTA 1-2 — each as `- [ ]` checkboxes for the founder to pick. The Visual block is optional (see below).

````markdown
## <NN> — <email name>

**Trigger:** <email-specific trigger from the outline>
**Target metric:** <email-level or sequence-level metric>

**Subject (pick one):**

- [ ] <variant 1>
- [ ] <variant 2>
- [ ] <variant 3>

**Preheader (pick one):**

- [ ] <variant 1>
- [ ] <variant 2>

**Body:**

<body prose at the sequence-type-driven length from the Body length lookup>

**Visual:**

- **Prompt:** <tool-agnostic image-generation prompt; brand-calibrated; persona-aware scene; add a play-button overlay if it is a video thumbnail>
- **Alt text:** <accessible description>
- **HTML embed:**

  ```html
  <a href="<VIDEO_URL>" target="_blank" rel="noopener">
    <img src="<THUMBNAIL_URL>" alt="..." width="600" style="display:block; max-width:100%; height:auto; border:0;" />
  </a>
  ```

**CTA (pick one):**

- [ ] <variant 1>
- [ ] <variant 2>
````

**Visual block is optional.** Emit it only for an email where a visual genuinely strengthens the message (e.g. a demo-video thumbnail, a product screenshot). Most emails have no Visual block. Do not key it off keywords — use judgment per email. No image files are ever written; the skill emits prompts and an embed template only.

## Idempotency & safety

- **Output directory** is `email-sequences/` in the working directory; create it if absent.
- **Filename** is `<slug>-<YYYY-MM-DD>.md`. Derive the slug from the sequence name (Unicode-aware):
    1. Apply Unicode NFC normalization.
    2. Lowercase (Unicode-aware).
    3. Replace whitespace, dots, and underscores with single hyphens.
    4. Strip characters that are not Unicode letters, Unicode digits, or hyphens.
    5. Collapse consecutive hyphens into one.
    6. Trim leading and trailing hyphens.
    - Accents are preserved (e.g. `Activación early` -> `activación-early`). Do not strip accents to ASCII.
- **Never overwrite.** Before writing, `Glob` `email-sequences/<slug>-<YYYY-MM-DD>*.md` and count existing matches. If `<slug>-<YYYY-MM-DD>.md` already exists, write the new draft to `<slug>-<YYYY-MM-DD>-<N>.md`, where `<N>` is the next free integer ≥ 2 (e.g. first collision -> `-2`, second -> `-3`). A count-based suffix is used rather than a timestamp because the skill has no Bash and cannot read the wall clock. The existing file is never modified by a draft run.
- **No binary files** are written — prompts and HTML embed templates only (v1 does not call image-generation APIs).
- **Consolidate is idempotent** for picked variants: running it twice on the same file with the same picks yields a byte-identical file.
- **Consolidate preserves prose edits** verbatim — it only collapses variant blocks and resolves ticked OQ blocks.

Document any hard-fail or non-default path clearly. Never silently overwrite, never silently skip a sequence.
