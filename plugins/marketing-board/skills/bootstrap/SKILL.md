---
name: marketing-board-bootstrap
description: Scaffold the marketing-board knowledge base (`agent_docs/marketing/`) for this project. Creates INTERVIEW.md; `--consolidate` produces 6 files. Explicit invocation only — `/marketing-board:bootstrap`.
disable-model-invocation: true
---

# Marketing-Board Bootstrap

Scaffolds the per-project marketing knowledge base that `/marketing-board` reads before every deliberation. Two phases:

1. **Scaffold** — create `agent_docs/marketing/INTERVIEW.md`, a guided interview covering product, audience, competition, business, distribution, and constraints.
2. **Consolidate** — parse the filled interview and write six knowledge-base files (`product.md`, `audience.md`, `competition.md`, `business.md`, `distribution.md`, `constraints.md`).

## When to use

Run this once per SaaS project, **before the first `/marketing-board` deliberation**. The deliberation command hard-fails until the knowledge base exists with all required `##` sections.

This skill never auto-fires. It runs only when the user types `/marketing-board:bootstrap` (optionally with a flag).

## Flag handling

Parse `$ARGUMENTS` first. Route per this table:

| Invocation                                         | Pre-condition                              | Action                                                                                |
| -------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------- |
| `/marketing-board:bootstrap`                       | `agent_docs/marketing/` does **not** exist | Run **Scaffold flow** (create directory + INTERVIEW.md)                               |
| `/marketing-board:bootstrap`                       | `agent_docs/marketing/` already exists     | Abort with the exact message in **Abort messages** below                              |
| `/marketing-board:bootstrap --reset`               | (any)                                      | Overwrite `INTERVIEW.md` with a fresh scaffold; **do NOT touch** the six target files |
| `/marketing-board:bootstrap --consolidate`         | `INTERVIEW.md` exists                      | Run **Consolidate flow** (write/overwrite the six target files)                       |
| `/marketing-board:bootstrap --consolidate`         | `INTERVIEW.md` does not exist              | Abort with the "INTERVIEW.md not found" message                                       |
| `/marketing-board:bootstrap --lang <code>`         | (any)                                      | Abort — `--lang` is a v1.1 feature (see **Abort messages**)                           |
| `/marketing-board:bootstrap --reset --consolidate` | (any)                                      | Abort — contradictory intent. User must pick one                                      |

**Abort messages (verbatim):**

| Condition                              | Message                                                                                                                                 |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Knowledge base already exists, no flag | `agent_docs/marketing/ already exists. Use --reset to overwrite INTERVIEW.md, or edit it directly and run --consolidate.`               |
| `--consolidate` but no INTERVIEW.md    | `INTERVIEW.md not found. Run /marketing-board:bootstrap first.`                                                                         |
| `--lang <any>`                         | `--lang is not available in v1. Bootstrap ships English-only; non-English support arrives in v1.1.`                                     |
| `--reset --consolidate`                | `--reset and --consolidate are mutually exclusive. Pick one: --reset rebuilds INTERVIEW.md; --consolidate writes the six target files.` |

## Scaffold flow

1. Verify `agent_docs/marketing/` does not exist. Skip this check entirely when `--reset` was passed.
2. Create the directory `agent_docs/marketing/` in the working directory if it does not already exist.
3. Read the template from `${CLAUDE_PLUGIN_ROOT}/skills/bootstrap/templates/interview.md`.
4. Write the template to `agent_docs/marketing/INTERVIEW.md` verbatim — no rewording, no LLM transformation. v1 is English-only.
5. Replace the placeholder `<product name>` in the title with the working directory's basename, kebab-cased (e.g. `empleo-digital`). If you can't determine the product name from the path, leave the placeholder.
6. Update the "Generated YYYY-MM-DD" line to today's date.
7. Report to the user:
    - The path created
    - That they should now fill `INTERVIEW.md` (mention they can ask Claude to interview them section-by-section using the hint at each section's top)
    - That `/marketing-board:bootstrap --consolidate` will produce the six target files when they're done

`--reset` runs the same flow (with step 1 skipped per above) and overwrites `INTERVIEW.md` if it exists. `--reset` **never** touches the six target files — only `INTERVIEW.md`.

## Consolidate flow

Two-pass hybrid engine. Pass 1 is deterministic; pass 2 uses an LLM only on free-text prose.

### Pass 1 — Deterministic parse

Walk `agent_docs/marketing/INTERVIEW.md` and extract, **byte-verbatim**:

- Checkbox state (`- [x]` and `- [ ]` lines)
- Tables (full rows)
- Bullet lists and numbered lists
- Code blocks
- `[skipped: <reason>]` markers

Do not invoke any model for pass-1 content. The same INTERVIEW.md fed in twice must produce byte-identical pass-1 output.

For each section in INTERVIEW.md (Section 1 → `product.md`, Section 2 → `audience.md`, etc.), map the Q-IDs to the target file's required `##` sections using the mapping below.

| Section         | Output file       | Q-ID → section mapping                                                                                                                                                                                                           |
| --------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Product      | `product.md`      | Q1.1 → `## Product description` · Q1.2 → `## Category` · Q1.3 → `## Value propositions` · Q1.4 → `## Positioning statement` · Q1.5 → `## Brand voice & tone`                                                                     |
| 2. Audience     | `audience.md`     | Q2.1 → `## ICP` · Q2.2/Q2.3/Q2.4 → `## Personas` · Q2.5 → `## Anti-personas` · Q2.6 → `## Attention map` · Q2.7 → `## Vocabulary` · Q2.8 → `## Triggers to buy`                                                                  |
| 3. Competition  | `competition.md`  | Q3.1 → `## Alternatives` · Q3.2 → `## How we differ` · Q3.3 → `## Category conventions to inherit` · Q3.4 → `## Category conventions to break` · (Q3.5, Q3.6 inform but do not own sections — fold into nearest related section) |
| 4. Business     | `business.md`     | **Q4.3 → frontmatter `lp_url`** (single-line URL; written when non-empty) · Q4.1+Q4.2 → `## Pricing tiers` · Q4.4 → `## Funnel state` · Q4.5 → `## Budget envelope` · Q4.6 → `## Timeline` · Q4.7 → `## Geographic focus`        |
| 5. Distribution | `distribution.md` | Q5.1 → `## Owned channels` · Q5.2 → `## Content inventory` · Q5.3 → `## Founder communities` · Q5.4 → `## Aspirational influencers` · Q5.5 → `## Past PR` · (Q5.6 informs but does not own a section)                            |
| 6. Constraints  | `constraints.md`  | Q6.1 → `## Legal / compliance` · Q6.2 → `## Founder risk tolerance` · Q6.3 → `## Past failed experiments` · Q6.4 → `## Unfair advantages` · Q6.5 → `## Regulatory landmines`                                                     |

For any question marked `[skipped: <reason>]` or left with the placeholder `> _Your answer here..._` still in place, place this callout in the corresponding section:

```markdown
<!-- TODO: skipped during bootstrap — <reason or "unanswered"> -->
```

### Frontmatter writing (per TDD 00 Frontmatter convention)

Some Q-IDs map to YAML frontmatter at the **top of the file**, not to a body `##` section. When the mapping table above marks a Q-ID as `→ frontmatter <key>`:

- Emit a YAML frontmatter block (`---\n<key>: <value>\n---`) at the very top of the target file, before the `# <Title>` H1, followed by a blank line.
- Only emit the block when at least one mapped frontmatter field has a non-empty answer. A blank or skipped LP URL produces NO frontmatter block (never write an empty `lp_url:` key).
- If a file has multiple frontmatter fields (none in v1, but the convention extends), order keys alphabetically for byte-identical idempotency.

Example for `business.md` when Q4.3 = `https://empleo.digital`:

```markdown
---
lp_url: https://empleo.digital
---

# Business

> Consolidated from `INTERVIEW.md`…
```

### Pass 2 — LLM polish on free-text prose only

Only the *free-text prose* answers from pass 1 go through Claude (you, in this session). For each prose answer:

- **PR-01 — No invented facts.** Preserve the user's facts verbatim. No inferred additions or removals.
- **PR-02 — Normalize voice.** Capitalization, punctuation, sentence flow — without changing meaning.
- **PR-03 — Do not infer skipped answers.** If marked skipped/unanswered, leave the TODO callout; do not fabricate.
- **PR-04 — Preserve language.** If the user wrote in Spanish, the output stays Spanish. Do not "correct" non-English answers into English — this applies even though v1 is English-only at the template level (a user is free to write their answers in any language inside the English template).
- **PR-05 — Enforce detail-level cap by condensation.** If the user wrote five sentences where the question asked for one, condense during polish while preserving meaning. Do **not** silently truncate. Do **not** warn-and-keep — condense.

Pass 2 output is *not* required to be byte-identical run-to-run, but MUST be semantically equivalent.

### Hand-edit preservation rule

Before writing each target file, if the file already exists:

1. Read it.
2. Identify any `##` sections in the existing file whose headers are **not** in the required-section list for that file.
3. Preserve those user-added sections verbatim — re-attach them at the bottom of the regenerated file, after the required sections.

Hand edits **inside** required sections are overwritten silently. Git is the safety net. This is documented behavior, not a bug.

### After writing all six files

1. Confirm each required `##` section appears in its target file (presence check, not content quality).
2. Report to the user:
    - The six files written, with paths
    - Any `<!-- TODO: -->` callouts that remain (skipped or unanswered questions)
    - One sentence: "Now run `/marketing-board <brief>` to convene the board."

Do **not** emit a freshness diff. Git already does that.

## Idempotency & safety

- **Default invocation aborts on existing knowledge base.** No accidental overwrites.
- **`--reset` only touches INTERVIEW.md.** The six target files survive a reset, so a user can experiment with reformulating questions without losing consolidated outputs.
- **`--consolidate` is idempotent for pass-1 content.** Structured fields (checkboxes, tables, lists, code blocks) are byte-identical across runs.
- **Pass-2 prose may differ across runs**, but never in factual content (PR-01).
- **User-added `##` sections in target files survive consolidate re-runs verbatim.**
- **Hand edits inside required sections do NOT survive consolidate.** INTERVIEW.md is the source of truth for required sections; the user can edit INTERVIEW.md and re-consolidate to update them.

Document any abort or non-default flow clearly to the user. Never silently overwrite.
