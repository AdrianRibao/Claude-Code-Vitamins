# TDD — Marketing Board (Master)

| Field          | Value                                                                                                                |
| -------------- | -------------------------------------------------------------------------------------------------------------------- |
| Type           | Master TDD                                                                                                           |
| Status         | v1.0 — consolidated, OQs resolved                                                                                    |
| Owner          | Adrián Ribao                                                                                                         |
| Created        | 2026-05-20                                                                                                           |
| Parent PRD     | [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md) |
| Pattern source | [`plugins/ceo-board`](../../../plugins/ceo-board)                                                                    |
| Mechanism      | [Claude Code subagents](https://code.claude.com/docs/en/sub-agents)                                                  |

______________________________________________________________________

## Overview

This master TDD owns the **cross-cutting contracts** for the marketing-board plugin: file layout, naming conventions, knowledge-base file shapes, plugin metadata. Three child TDDs cover the feature-level work:

| Doc                                                   | Scope                                                                                                                                                    |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [01-deliberation](01-marketing-board-deliberation.md) | `/marketing-board` command, 8 subagents, synthesizer, Marketing Plan Memo                                                                                |
| [02-bootstrap](02-marketing-board-bootstrap.md)       | `/marketing-board:bootstrap` skill, INTERVIEW.md schema, knowledge-base generation                                                                       |
| [03-email-sequence-skill](03-email-sequence-skill.md) | `/marketing-board:email-sequence` skill — drafts ESP-paste-ready email sequences from a saved memo with variant checkboxes and `--consolidate` iteration |

The PRD has been consolidated (v1.0); these TDDs translate v1 requirements into testable contracts. No implementation code in TDDs — signatures and shapes only.

## Plugin file structure

```
plugins/marketing-board/
├── .claude-plugin/
│   └── plugin.json                 # name, version, description, author
├── README.md                        # seat composition, briefing tips, design rationale
├── commands/
│   └── marketing-board.md          # /marketing-board <brief> (TDD 01)
├── agents/
│   ├── ethnographer.md
│   ├── storyteller.md
│   ├── channel-strategist.md
│   ├── producer.md
│   ├── funnel-engineer.md
│   ├── community-pr-lead.md
│   ├── moonshot.md
│   └── contrarian.md
├── reference/
│   └── synthesizer-prompt.md       # Loaded by commands/marketing-board.md at step 5 (TDD 01 OQ-D-05)
└── skills/
    ├── bootstrap/
    │   ├── SKILL.md                # /marketing-board:bootstrap (TDD 02)
    │   └── templates/
    │       └── interview.md        # Single English source; LLM translates at runtime per --lang (OQ-M-03)
    └── email-sequence/
        └── SKILL.md                # /marketing-board:email-sequence (TDD 03)
```

Marketplace registration: append a row in `.claude-plugin/marketplace.json` under `plugins`.

## Naming conventions

| Artifact                  | Convention                              | Example                                |
| ------------------------- | --------------------------------------- | -------------------------------------- |
| Agent files               | `<slug>.md`                             | `community-pr-lead.md`                 |
| Agent slugs               | lowercase, kebab-case                   | `community-pr-lead`, `funnel-engineer` |
| Agent `name:` frontmatter | identical to slug                       | `name: community-pr-lead`              |
| Knowledge-base files      | `<topic>.md`, kebab-case                | `audience.md`, `competition.md`        |
| Plugin version            | semver                                  | `0.1.0` initial                        |
| Slash command name        | matches plugin (CLAUDE.md convention)   | `/marketing-board`                     |
| Saved memo filename       | `<product>-<YYYY-MM-DD>.md`             | `empleo-digital-2026-05-20.md`         |
| Memo save directory       | `marketing-plans/` in working directory | `~/proyectos/empleo/marketing-plans/`  |

## Plugin metadata (`plugin.json`)

| Field         | Value                                                                                     |
| ------------- | ----------------------------------------------------------------------------------------- |
| `name`        | `marketing-board`                                                                         |
| `version`     | `0.1.0` (bump on every change per repo CLAUDE.md)                                         |
| `description` | "Eight-seat marketing deliberation board for SaaS launches. Mirrors `ceo-board` pattern." |
| `author`      | `Adrián Ribao Martínez`                                                                   |

## Knowledge-base file contracts (cross-cutting)

These six files MUST exist at `agent_docs/marketing/` in the SaaS project root before `/marketing-board` runs. Bootstrap (TDD 02) produces them. Agents (TDD 01) consume them. Each file is a **structured Markdown document** — section headings are part of the contract; section bodies are the user's content.

### `product.md`

| Required section           | Body shape                                                                         |
| -------------------------- | ---------------------------------------------------------------------------------- |
| `## Product description`   | 1-3 sentences, prose                                                               |
| `## Category`              | 1 sentence; "X tool for Y" form                                                    |
| `## Value propositions`    | Bullet list, 3-5 items; each item: bold headline + 1-line proof point              |
| `## Positioning statement` | 1 sentence; "For [audience], [product] is the [category] that [unique value]" form |
| `## Brand voice & tone`    | 3-7 adjective bullets + 1 example sentence in voice                                |

### `audience.md`

| Required section     | Body shape                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `## ICP`             | 3-5 dimensions: segment, role, company size, geography, sophistication                     |
| `## Personas`        | 1-3 named personas, each with: JTBD (1 line), top 3 pains, day-in-the-life (2-3 sentences) |
| `## Anti-personas`   | Bullet list, 2-4 items; who is explicitly NOT the target and why                           |
| `## Attention map`   | Table: channel \| platform \| signal strength (high/med/low)                               |
| `## Vocabulary`      | Two columns: terms-they-use, terms-to-avoid                                                |
| `## Triggers to buy` | Bullet list, 3-5 events that prompt purchase consideration                                 |

### `competition.md`

| Required section                     | Body shape                                                                 |
| ------------------------------------ | -------------------------------------------------------------------------- |
| `## Alternatives`                    | 3-5 alternatives (incl. status quo / DIY); each: name + 1-line description |
| `## How we differ`                   | Table: alternative \| our advantage \| their advantage                     |
| `## Category conventions to inherit` | Bullet list of conventions worth following                                 |
| `## Category conventions to break`   | Bullet list of conventions worth breaking + why                            |

### `business.md`

| Required section      | Body shape                                                             |
| --------------------- | ---------------------------------------------------------------------- |
| `## Pricing tiers`    | Table: tier \| price \| key inclusions \| target persona               |
| `## Funnel state`     | LP URL, signup flow description (prose), known conversion rates if any |
| `## Budget envelope`  | Total budget + per-month allocation if known; "TBD" allowed            |
| `## Timeline`         | Launch date or window; runway in months                                |
| `## Geographic focus` | Primary markets (countries / language regions); locale code            |

### `distribution.md`

| Required section              | Body shape                                          |
| ----------------------------- | --------------------------------------------------- |
| `## Owned channels`           | Bullet list with platform + audience size           |
| `## Content inventory`        | Table: asset type \| count \| recency               |
| `## Founder communities`      | Bullet list of communities the founder is active in |
| `## Aspirational influencers` | Table: name \| platform \| audience \| relevance    |
| `## Past PR`                  | Bullet list of past mentions; "None" allowed        |

### `constraints.md`

| Required section             | Body shape                                                           |
| ---------------------------- | -------------------------------------------------------------------- |
| `## Legal / compliance`      | Bullet list of constraints (GDPR, sector regs, etc.); "None" allowed |
| `## Founder risk tolerance`  | One of: conservative / moderate / aggressive + 1 sentence rationale  |
| `## Past failed experiments` | Bullet list; "None" allowed                                          |
| `## Unfair advantages`       | Bullet list, 1-5 items; specific, not generic                        |
| `## Regulatory landmines`    | Bullet list of category-specific risks; "None" allowed               |

### Section-presence rule (extensible — OQ-M-02)

A file is "valid" if **all required sections are present as `##` headers**. Empty bodies are acceptable when prefixed with a `<!-- TODO: <reason> -->` callout. The deliberation command's frame step (TDD 01, FR-15) does a presence check, not a content quality check.

**Extensibility**: users MAY add additional `##` sections beyond the required set (e.g. `## Notes on Q1 2026 pivot` in `audience.md`). Extras are:

- **Preserved verbatim** through `--consolidate` re-runs (bootstrap must not strip them).
- **Available to agents** — any agent's `## Context loading` step reads the whole file, not just required sections.
- **Not validated** — extras are user scratchpad; no shape rules apply.

This makes the knowledge base a living document the user can extend without forking the plugin.

## Cross-document conventions

### Markdown style

- Tables for any 2+ row structured data
- Bullet lists for unordered enumerations
- `## Heading` style for required sections (not `#`)
- No code blocks > 5 lines in agent prompts or generated files
- Use Obsidian/GitHub-compatible markdown (no MDX, no JSX)

### Locale handling (architecture per OQ-M-03; v1 scope per TDD 02 OQ-B-06)

**Architecture (locked, applies in v1 and v1.1):**

- **Single source of truth**: the bootstrap skill ships **one** English template at `skills/bootstrap/templates/interview.md`. There are no per-language template files.
- **Knowledge-base files** inherit the language the user actually wrote in (the consolidator preserves it per PR-04 in TDD 02).
- **Agent system prompts**: English (Claude operates multilingually regardless of prompt language).
- **Memo language**: follows the brief language.

**v1 ship scope (per TDD 02 OQ-B-06):**

- Bootstrap is **English-only**. The `--lang <code>` flag is rejected with: `"--lang is not available in v1. Bootstrap ships English-only; non-English support arrives in v1.1."`
- Hard-fail messages from `/marketing-board` do not mention `--lang`.
- Users can still *write their answers* in any language inside the English INTERVIEW.md template; the consolidator's PR-04 preserves the language they used.

**v1.1 addition (already designed, not yet shipped):**

- The `--lang <code>` flag becomes active. Bootstrap uses Claude to translate the English template into the target language at runtime, preserving question IDs, render format, checkbox structures, and worked-example shapes.
- `--lang` accepts any ISO 639-1 code Claude can translate to (e.g. `en`, `es`, `pt`, `fr`, `de`). No allowlist; the LLM is the translator.
- v1.1 unlocks the flag without changing the template or knowledge-base files.

**Trade-off accepted**: runtime translation is less deterministic than static template files, but the translation prompt is constrained tightly (preserve IDs, structure, examples) and the cost of adding a new language drops to zero.

### Idempotency conventions

| Action                                     | Idempotent? | Notes                                                                                           |
| ------------------------------------------ | ----------- | ----------------------------------------------------------------------------------------------- |
| `/marketing-board:bootstrap` (no flag)     | No          | Aborts if `agent_docs/marketing/` exists                                                        |
| `/marketing-board:bootstrap --reset`       | No          | Destructive; overwrites INTERVIEW.md                                                            |
| `/marketing-board:bootstrap --consolidate` | **Yes**     | Same INTERVIEW.md → same output every time                                                      |
| `/marketing-board <brief>`                 | No          | Distinct deliberation each time; always writes a memo file (same-day re-run → `-HHMMSS` suffix) |
| `/marketing-board --consolidate [@memo]`   | **Yes**     | Same memo file + same in-file answers → same consolidated memo, written back in place           |

## Versioning rule

Per repo [CLAUDE.md](../../../CLAUDE.md): bump `plugin.json` `version` on **every** change inside the plugin. This includes:

- Agent prompt edits
- Command body edits
- Bootstrap template edits
- README edits
- New files added

Initial version: `0.1.0`. First public-ready release: `1.0.0`.

## Authorization

**N/A** — this is a personal Claude Code plugin invoked in the user's own working directory. No multi-user authorization layer. No data leaves the user's machine except via tool-mediated WebSearch/WebFetch calls, which inherit Claude Code's existing permissions.

## Testing requirements (cross-cutting)

| Category                        | Approach                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| **Manual integration**          | Run end-to-end on empleo.digital (acceptance test) and one second SaaS (reuse test)   |
| **Frontmatter validation**      | Lint each agent and the bootstrap skill file for required frontmatter fields          |
| **Section presence**            | Generated knowledge-base files contain all required `##` headers                      |
| **Idempotency**                 | `--consolidate` runs produce byte-identical output across invocations                 |
| **Hard-fail messaging**         | Missing knowledge-base files → exact hard-fail message (TDD 01, FR-15)                |
| **Prompt review (qualitative)** | Each agent's lens stays orthogonal; manual A/B review of memos for distinguishability |

No automated test framework dependency in v1 — manual acceptance tests + qualitative review. (Re-evaluate in v2 if the plugin starts receiving external contributions.)

## Related Documents

- **PRD**: [`specs/prds/marketing-board/00-marketing-board-master.md`](../../prds/marketing-board/00-marketing-board-master.md)
- **TDD 01** (Deliberation): [`01-marketing-board-deliberation.md`](01-marketing-board-deliberation.md)
- **TDD 02** (Bootstrap): [`02-marketing-board-bootstrap.md`](02-marketing-board-bootstrap.md)
- **Pattern source**: [`plugins/ceo-board/`](../../../plugins/ceo-board/)
- **Marketplace conventions**: [`CLAUDE.md`](../../../CLAUDE.md), [`README.md`](../../../README.md)
- **Subagent reference**: <https://code.claude.com/docs/en/sub-agents>

## Open Questions

*All questions resolved and integrated as of 2026-05-20.*

### Resolution log

| ID      | Question (short)                              | Resolution                                                                                                                                                                 | Integrated in                                                                          |
| ------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| OQ-M-01 | Community/PR Lead slug                        | `community-pr-lead` (explicit; matches PRD lineup name)                                                                                                                    | Naming conventions, file structure tree, agent file paths (TDD 01 follow-up needed)    |
| OQ-M-02 | Strict vs. extensible knowledge-base sections | **Extensible** — required sections enforced; extras preserved verbatim and visible to agents                                                                               | "Section-presence rule (extensible)" subsection                                        |
| OQ-M-03 | INTERVIEW.md template engine                  | **Single English template + LLM translation at runtime** — one `templates/interview.md`; `--lang <code>` accepts arbitrary ISO 639-1 codes                                 | File structure tree, "Locale handling (resolved)" subsection (TDD 02 follow-up needed) |
| OQ-M-04 | Memo save directory configurability           | **Hardcoded `marketing-plans/`** in working directory; revisit when a real conflict surfaces. (The memo is saved automatically — see PRD FR-20 / Architecture Decision 9.) | Naming conventions ("Memo save directory" row)                                         |
