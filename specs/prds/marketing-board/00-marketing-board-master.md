# PRD — Marketing Board (Master)

| Field          | Value                                                               |
| -------------- | ------------------------------------------------------------------- |
| Type           | Master PRD                                                          |
| Status         | v1.1 — always-save revision applied; TDDs + plugin built (v0.2.x)   |
| Owner          | Adrián Ribao                                                        |
| Created        | 2026-05-20                                                          |
| First test     | empleo.digital (Spanish-language job SaaS)                          |
| Pattern source | [`plugins/ceo-board`](../../../plugins/ceo-board) (sibling plugin)  |
| Mechanism      | [Claude Code subagents](https://code.claude.com/docs/en/sub-agents) |

______________________________________________________________________

## Problem Statement

A SaaS launch needs a marketing plan that has been pressure-tested from many angles — audience research, positioning, channel mix, content production, funnel design, risk analysis. A single Claude conversation collapses these perspectives into one voice, which loses the orthogonality that strategic deliberation depends on. Important angles get under-weighted or skipped entirely, and the resulting plans tend to be bland and over-broad.

The author is launching empleo.digital (Spanish-language job SaaS) and has more SaaS launches ahead. Each one needs a coherent marketing plan that has been forced to defend itself against eight independent, sharp lenses — and that the author can iterate against quickly.

The existing `ceo-board` plugin proves the multi-agent deliberation pattern works for strategic decisions; this PRD applies the same pattern to marketing strategy.

## Goals

| ID   | Goal                                                                                | Measure                                                                                                                             |
| ---- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| G-01 | Produce a coherent marketing plan from a SaaS brief in a single command invocation  | One synthesized plan per `/marketing-board` call                                                                                    |
| G-02 | Force orthogonal perspectives — eight distinct lenses that don't restate each other | Synthesizer can identify ≥4 genuinely conflicting positions per memo                                                                |
| G-03 | Surface risks and weakest assumptions before budget is committed                    | Contrarian report exists in every deliberation                                                                                      |
| G-04 | Reusable across SaaS products without modification                                  | Same plugin works on a second product post-empleo.digital                                                                           |
| G-05 | Mirror the proven `ceo-board` shape (command + agents + synthesizer in main thread) | Plugin structurally analogous to `ceo-board`; no auto-invocation                                                                    |
| G-06 | Lay the foundation for downstream production skills (v2+)                           | PRD architecture leaves room for child PRDs without restructuring v1                                                                |
| G-07 | Stable per-product context separated from per-deliberation brief                    | Each project has `agent_docs/marketing/*.md` knowledge base; agents load it automatically; brief carries only the specific decision |

## Non-Goals

- **Not** auto-generating finished marketing assets (emails, video scripts, ad copy). That belongs to v2+ production skills.
- **Not** executing campaigns (sending emails, posting ads, publishing content). The output is a plan, not an automation.
- **Not** replacing human marketing judgment. The plan is input to a human decision, not a verdict.
- **Not** tracking campaign performance or attribution. Output is forward-looking, not retrospective.
- **Not** locked to a single language/market — empleo.digital is the first test case, but the fleet operates on whatever locale the brief specifies.
- **Not** a CRM, ESP, or marketing-automation replacement.

## Target Users

### Primary: SaaS founder (the author)

- **Role**: Solo or small-team founder launching SaaS products
- **Context**: Strong technical/product chops, less marketing depth
- **Pain**: Generic LLM marketing prompts produce bland, over-broad plans missing critical angles
- **Need**: Pressure-tested strategy across multiple lenses, on demand, repeatable
- **Workflow**: Drafts a brief → invokes `/marketing-board` → reviews 8 reports + synthesized plan → iterates

### Secondary (future): Other SaaS founders via marketplace install

- The plugin is intended to be reusable; the author may share it via this marketplace
- Out of scope for v1 design decisions (no team/multi-user features)

## User Workflows

### Workflow 1 — Launch deliberation (primary)

1. User opens Claude Code in a project (e.g. `~/proyectos/empleo.digital`).
2. User types `/marketing-board <brief>` where `<brief>` describes the product, target market, current assets (LP, lists, audience signals), launch timeline, and any constraints.
3. The slash command's main-session script:
    - **Verifies** that `agent_docs/marketing/` exists with the six expected files (per FR-15). If anything is missing, hard-fails with a single prompt to run `/marketing-board:bootstrap` first. No partial-context fallback.
    - Frames the decision (restates the brief, flags missing context within the brief itself).
    - Dispatches the eight subagents in parallel via the Agent tool — each in its own fresh context window with the framed brief.
4. Each subagent runs independently in its own fresh context: loads the relevant product-context files from `agent_docs/marketing/` (see [Product Knowledge Base](#product-knowledge-base-agent_docsmarketing)), applies its lens to the framed brief, and returns a structured verdict. No agent reads another's work.
5. Main session synthesizes a single **Marketing Plan Memo** addressing convergent points, real disagreements, and the Contrarian's strongest objection.
6. **Deep-analysis pass for Open Questions.** Main session re-reads the synthesized memo plus the eight raw verdicts and surfaces unresolved decisions that would block execution — budget thresholds, channel sequencing calls, assets to ready before launch, accepted-risk tolerances, pricing/positioning calls still pending a human decision. Each OQ gets a unique ID, a clear question, why-it-matters rationale, possible answers, and a status. (Mirrors the PRD/TDD skill's Phase 2.)
7. The synthesized memo is **written to `marketing-plans/<product>-<YYYY-MM-DD>.md`** automatically (per FR-20). User receives: eight raw reports (in their respective subagent transcripts) + the **Marketing Plan Memo** in the conversation and on disk, with the **Open Questions** section at the end. User answers OQs by editing that memo file — ticking the `- [ ]` checkboxes, adding notes — then runs `/marketing-board --consolidate` (see Workflow 4) to fold answers into the memo.

**Estimated time:** 3–8 minutes per deliberation, depending on agent model selection and brief depth.

### Workflow 2 — Plan iteration

1. User reviews the synthesized memo, identifies weak sections or counter-evidence.
2. User edits the brief (tightens scope, adds constraints, specifies budget/timeline) and re-runs `/marketing-board`.
3. User compares plans across runs; commits to one.

### Workflow 3 — Single-seat consultation (secondary)

1. User wants depth on a specific angle (e.g. "I just need an Ethnographer pass on this LP").
2. User invokes one agent directly via `@agent-marketing-board:ethnographer` or natural-language delegation.
3. Single agent returns its report; no synthesis step.

This is a secondary workflow — primary value is the full deliberation.

### Workflow 4 — Memo consolidation (after answering Open Questions)

1. User opens the saved memo file (`marketing-plans/<product>-<YYYY-MM-DD>.md`, written automatically by Workflow 1) and answers the **Open Questions** section **in the file** — ticking the `- [ ]` checkbox on the chosen option, optionally adding a note.
2. User runs `/marketing-board --consolidate` (optionally with an explicit path, e.g. `@marketing-plans/empleo-digital-2026-05-20.md`; defaults to the most recently written memo file in `marketing-plans/`).
3. Main session reads the memo file, applies the answers marked in it into the relevant memo sections (`The plan I'm proposing`, `What I'm accepting as risk`, `Pre-launch checklist`, `First 30 days`), marks resolved OQs as `Resolved` with a one-line answer, tightens overlapping prose, and writes the updated memo **back to the same file**.
4. User gets a consolidated memo with fewer open decisions, ready for execution. (Mirrors the `/prd --consolidate` flow from the prd-writer skill.)

**Trigger conditions:** invoke when 3+ OQs have been answered, or any time before kicking off execution.

### Workflow 5 — Bootstrap product context (one-time setup, run before Workflow 1)

This is the **required** setup flow run once per SaaS project. Workflow 1 (deliberation) hard-fails if this hasn't been completed — the eight agents require the knowledge base to be authoritative. See [Product Knowledge Base](#product-knowledge-base-agent_docsmarketing) for the file contract.

1. User opens Claude Code in the SaaS project root and runs `/marketing-board:bootstrap`. (v1 ships an English-only template; the `--lang <code>` flag for localized templates is deferred to v1.1 per TDD 02 OQ-B-06.)
2. The skill scaffolds `agent_docs/marketing/INTERVIEW.md` — a single Markdown file with 6 sections (Product, Audience, Competition, Business, Distribution, Constraints). Each section contains:
    - A short header explaining what this section is for and why agents need it.
    - Questions with inline **explanations**, **at least one worked example**, and a **skip** marker for non-applicable questions.
    - Multiple-choice questions rendered as GitHub-flavored checkbox lists (`- [ ] Option A`) — user changes `[ ]` to `[x]` to select. Multi-select supported where appropriate.
    - Free-text fields with clear instructions on the expected level of detail (one sentence vs. one paragraph vs. bullet list).
    - An **"ask Claude to interview me"** hint at the top of each section, naming the natural-language prompt the user can paste to switch into a guided Q&A for that section (e.g. *"Interview me on the Audience section — ask one question at a time and update my answers in INTERVIEW.md."*). This gives template-first users an opt-in conversational mode without baking interactive flow into the skill itself.
3. User fills `INTERVIEW.md` — possibly over multiple sessions, possibly with Claude's help interactively per the embedded hint, possibly by asking Claude to read project artifacts (e.g. *"draft answers from the LP at empleo.digital, and I'll review"*).
4. User runs `/marketing-board:bootstrap --consolidate`.
5. The skill reads the filled `INTERVIEW.md` and produces six target files under `agent_docs/marketing/`:
    - `product.md` — description, category, value props, positioning draft, brand voice
    - `audience.md` — ICP, 1-3 buyer personas, anti-personas, attention map, vocabulary
    - `competition.md` — 3-5 alternatives, how-we-differ, category conventions
    - `business.md` — pricing, funnel state, conversion data, budget envelope, timeline
    - `distribution.md` — owned channels, content inventory, founder communities, aspirational influencers
    - `constraints.md` — legal/compliance, founder risk tolerance, past failures, unfair advantages
6. The skill preserves user selections verbatim (checkbox state, structured fields) and uses LLM rewriting only to polish free-text prose into the target file's voice. Incomplete or skipped sections are marked with a `<!-- TODO: -->` callout in the output so the user knows what still needs attention.
7. Maintenance: after bootstrap, the user can edit `agent_docs/marketing/*.md` files directly as the product evolves. Re-running bootstrap is only needed for a major reframing.

**Estimated time:** 30-90 minutes to fill `INTERVIEW.md` thoughtfully on the first pass; 5 minutes to consolidate.

## Requirements

### Functional (FR)

| ID    | Requirement                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Priority |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| FR-01 | Plugin provides a slash command (default name: `/marketing-board`) that accepts a free-form brief as `$ARGUMENTS`                                                                                                                                                                                                                                                                                                                                                                                                                                     | P0       |
| FR-02 | Plugin defines eight independent subagents under `agents/`, each in its own `.md` file with YAML frontmatter and a focused system prompt                                                                                                                                                                                                                                                                                                                                                                                                              | P0       |
| FR-03 | Each subagent's `description` field is sharp and lens-specific (no overlap), so Claude can also route to it via natural language or `@agent-` mention                                                                                                                                                                                                                                                                                                                                                                                                 | P0       |
| FR-04 | Subagents run in parallel — main session dispatches all eight concurrently in a single Agent-tool fan-out, no agent reads another's output before synthesis                                                                                                                                                                                                                                                                                                                                                                                           | P0       |
| FR-05 | The slash command instructs the main session to synthesize a **Marketing Plan Memo** with a consistent structure (sections defined in the command file)                                                                                                                                                                                                                                                                                                                                                                                               | P0       |
| FR-06 | The Contrarian seat is always included and always invoked — it cannot be opted out                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-07 | The plugin works across SaaS products without modification — locale, audience, and channel context come from the brief, not from agent definitions                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-08 | Auto-invocation of the full board is disabled — only the explicit slash command (or explicit natural-language convening) triggers the deliberation                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-09 | Each subagent has tool access scoped to research and reading only (`Read, Grep, Glob, WebSearch, WebFetch`) — no Write/Edit/Bash                                                                                                                                                                                                                                                                                                                                                                                                                      | P0       |
| FR-10 | The slash command frame step uses repo-aware tools (Read/Grep/Glob) to pull in product context (LP copy, existing docs) when the brief references them                                                                                                                                                                                                                                                                                                                                                                                                | P1       |
| FR-11 | Plugin includes a README documenting seat composition, decision rationale, how to brief well (with a recommended brief template to copy), and the "don't outsource the decision" disclaimer                                                                                                                                                                                                                                                                                                                                                           | P0       |
| FR-12 | Plugin metadata (`plugin.json`) is registered in `.claude-plugin/marketplace.json`                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | P0       |
| FR-13 | After synthesis (FR-05), main session generates an **Open Questions** section via deep analysis — unresolved decisions that would block execution, each with unique ID, question, why-it-matters, possible answers, and status (Open / Deferred). Mirrors PRD/TDD skill Phase 2                                                                                                                                                                                                                                                                       | P0       |
| FR-14 | `/marketing-board --consolidate [@path]` reads the saved memo file, applies the OQ answers the user marked **in that file** (ticked checkboxes + inline notes) into the memo's relevant sections, marks resolved OQs, tightens prose, and writes the updated memo back to the same file. Mirrors `/prd --consolidate` from the prd-writer skill                                                                                                                                                                                                       | P1       |
| FR-15 | Each subagent **requires** the relevant files from `agent_docs/marketing/` at deliberation start. The slash command's frame step (FR-10) checks for the knowledge base **before** dispatching agents; if `agent_docs/marketing/` is missing or any of the six expected files is absent, the command **hard-fails** with a single actionable message: *"Bootstrap your product context first: `/marketing-board:bootstrap`."* No brief-only fallback in v1                                                                                             | P0       |
| FR-16 | Plugin includes a `bootstrap` skill (`/marketing-board:bootstrap`) that scaffolds `agent_docs/marketing/INTERVIEW.md` — a single Markdown file with six sections (Product, Audience, Competition, Business, Distribution, Constraints) covering the inputs each seat needs. v1 ships an English-only template; the `--lang <code>` flag for runtime-localized templates is deferred to v1.1 (per TDD 02 OQ-B-06)                                                                                                                                      | P0       |
| FR-17 | `INTERVIEW.md` is designed for easy filling: every question has (a) the question itself, (b) a one-line **why** rationale, (c) at least one concrete worked **example**, (d) explicit instructions on detail level (sentence / paragraph / list), (e) a clearly marked **skip** option for non-applicable questions. Multiple-choice questions use GitHub-flavored checkbox lists (`- [ ] Option`) so selection is a single character flip. Each section opens with an *"ask Claude to interview me"* prompt hint enabling opt-in conversational fill | P0       |
| FR-18 | `/marketing-board:bootstrap --consolidate` reads filled `INTERVIEW.md` and produces six target files (`product.md`, `audience.md`, `competition.md`, `business.md`, `distribution.md`, `constraints.md`) under `agent_docs/marketing/`. Structured fields (checkbox selections, tables) preserved verbatim; free-text prose polished into target-file voice. Incomplete sections flagged with `<!-- TODO: -->` callouts. Consolidation is idempotent — safe to re-run any time the user updates `INTERVIEW.md`                                        | P0       |
| FR-19 | `/marketing-board:bootstrap` (no flag) aborts with a notice if `agent_docs/marketing/` already exists; full re-scaffolding requires explicit `--reset` flag. Section-level updates happen by editing `INTERVIEW.md` directly and re-running `--consolidate` (which is always idempotent per FR-18). The dangerous-action default is to refuse, never overwrite                                                                                                                                                                                        | P1       |
| FR-20 | Every `/marketing-board` deliberation **automatically writes** the synthesized memo to `marketing-plans/<product>-<YYYY-MM-DD>.md` in the working directory (creating `marketing-plans/` if absent; a same-day second run gets a `-<HHMMSS>` suffix so an annotated file is never clobbered). The saved file is the durable artifact the user annotates and iterates via `--consolidate`. There is no opt-out flag — persistence is always on                                                                                                         | P1       |

### Non-Functional (NFR)

| ID     | Requirement                                                                                                                                                                                                                                                | Priority |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| NFR-01 | Each agent's prompt is tightly scoped to its lens — no scope sprawl across seats                                                                                                                                                                           | P0       |
| NFR-02 | Plugin versioning follows marketplace convention: bump `plugin.json` `version` on every change inside the plugin (per [CLAUDE.md][1])                                                                                                                      | P0       |
| NFR-03 | Plugin uses **only** Anthropic's documented subagent mechanism — no custom orchestration scripts, no MCP-dependent agents                                                                                                                                  | P0       |
| NFR-04 | Synthesizer logic lives in `commands/<name>.md`, not in a separate "synthesizer agent" (subagents cannot spawn subagents per the docs)                                                                                                                     | P0       |
| NFR-05 | Default models per seat: `opus` for **Storyteller**, **Moonshot**, **Contrarian** (narrative subtlety, asymmetric reasoning, sharp skepticism); `sonnet` for the other five. Documented as a single-line frontmatter field per agent so users can override | P1       |
| NFR-06 | Synthesizer prompt is editable and versioned independently — it's the load-bearing artifact for output quality                                                                                                                                             | P0       |
| NFR-07 | Each agent's output uses a fixed Markdown structure so the synthesizer can parse verdicts reliably                                                                                                                                                         | P1       |

## The Eight Seats (v1 lineup)

Each seat is a single Markdown file under `plugins/marketing-board/agents/`, structurally analogous to `plugins/ceo-board/agents/*.md`.

| Seat                   | Lens                                                                                                                          | Output                                                                                 | Default model |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------- |
| **Ethnographer**       | Audience intelligence — who buys, where they pay attention, what vocabulary they use, what pain triggers buying               | ICP profile, persona variants, attention map (communities, channels)                   | sonnet        |
| **Storyteller**        | Positioning & narrative — category fit, vs. status quo, message hierarchy, hero narrative                                     | Positioning statement, message house, taglines                                         | **opus**      |
| **Channel Strategist** | Demand generation — paid/organic/owned/earned mix, budget allocation, channel sequencing, CAC expectations                    | Channel plan with timeline & budget, paid creative briefs                              | sonnet        |
| **Producer**           | Content & creative production — what gets made: articles, video, email, social                                                | Editorial calendar, asset list, content briefs, video shotlists                        | sonnet        |
| **Funnel Engineer**    | Conversion & lifecycle — LP→trial→paid path, problem-solution sequences, onboarding, retention loops, A/B priorities          | Funnel map, email sequence outlines, conversion-leverage audit                         | sonnet        |
| **Community/PR Lead**  | Earned attention & launch sequence — who do we want talking about us, which communities/influencers matter, launch-day timing | Communities map, ambassadors/creators list, launch-day sequence, PR pitch angles       | sonnet        |
| **Moonshot**           | Asymmetric bet & 10x reframing — viral concept, unconventional partnership, channel that breaks the rules                     | Reframing memo with 1–3 asymmetric bets, expected payoff, what each would cost to test | **opus**      |
| **Contrarian**         | Pre-mortem & skeptic — weakest assumptions, failure modes, what to pre-test, kill criteria                                    | Risk register, premortem memo, what-would-change-my-mind list                          | **opus**      |

**Why these eight:** the first six are the core marketing lenses (audience, positioning, demand, content, conversion, skeptic). **Community/PR Lead** adds the earned-attention / launch-sequence lens that Channel Strategist's paid/organic/owned/earned framing doesn't cover at the relationship level — critical for launches where momentum comes from communities, influencers, and timing rather than budget. **Moonshot** pairs with Contrarian: one stretches the upside (10x reframings, asymmetric bets), the other pressure-tests the downside. Together they prevent the deliberation from collapsing into incrementalism.

**Tool access (per FR-09):** All eight get `Read, Grep, Glob, WebSearch, WebFetch`. None get `Write`, `Edit`, or `Bash`. Rationale: agents produce text reports, not artifacts on disk; web access lets Ethnographer find current communities, Community/PR Lead find live influencer/community signal, and Contrarian cite failed-launch base rates.

## Product Knowledge Base (`agent_docs/marketing/`)

Each SaaS project bootstrapped with `/marketing-board:bootstrap` (see [Workflow 5](#workflow-5--bootstrap-product-context-one-time-setup-run-before-workflow-1)) gets a stable, file-based knowledge base that every deliberation reads from. This separates **what the product is** (rarely changes) from **what we're deciding right now** (per-brief). Without it, the eight agents must reinvent ICPs and channel histories on every run, which defeats the orthogonality the deliberation pattern depends on.

### File contract — what each agent reads

| File              | Owns                                                                                                                   | Primary readers                                           |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `product.md`      | Product description, category, value props with proof points, positioning draft, brand voice & tone                    | All seats (everyone needs the product baseline)           |
| `audience.md`     | ICP definition, 1-3 buyer personas (JTBD, pains, day-in-the-life), anti-personas, attention map, buyer vocabulary      | Ethnographer (primary), Storyteller, Producer             |
| `competition.md`  | 3-5 alternatives (incl. status quo / DIY), how-we-differ, category conventions to inherit or break                     | Storyteller (primary), Contrarian, Moonshot               |
| `business.md`     | Pricing tiers, current funnel state, conversion data (if any), budget envelope, timeline, geographic focus             | Channel Strategist (primary), Funnel Engineer             |
| `distribution.md` | Owned channels (LP, lists, social), existing content inventory, founder communities, aspirational influencers, past PR | Channel Strategist, Community/PR Lead (primary), Producer |
| `constraints.md`  | Legal/compliance constraints, founder risk tolerance, past failed experiments, unfair advantages, regulatory landmines | Contrarian (primary), Moonshot, all others as needed      |

### Agents own the contract; bootstrap owns the materialization

This separation matters: agents declare which files they expect; the bootstrap skill is responsible for materializing those files. The agents can be re-tuned without touching the bootstrap skill; the bootstrap skill can change its interview UX without breaking the agents. The file contract above is the seam between the two.

### Knowledge base is required (no fallback)

When `agent_docs/marketing/` is missing or any of the six expected files is absent (per OQ-14 resolution):

- The slash command **hard-fails** at the frame step (before any agent is dispatched).
- The error message is a single line with the fix: *"Bootstrap your product context first: `/marketing-board:bootstrap`."*
- No brief-only fallback in v1. Brief-only output is misleading — it looks like an answer but is fundamentally ungrounded; the deliberation pattern depends on the knowledge base being authoritative.
- Bootstrap is **required** before first deliberation in any new project. Workflow 5 must run before Workflow 1.

## Marketing Plan Memo — Output Structure (v1)

The synthesizer (executed in the main session per NFR-04) produces a memo with this structure. The slash command's body locks this format so output is predictable across runs.

**Attribution convention** (per OQ-10 resolution): the memo reads as a single voice (the synthesizer's plan), **except** in the *Where the board disagrees* section where positions are explicitly named with the seats that took them. Single-voice plan, attributed disagreements.

```markdown
# Marketing Plan: [Product]

## Brief recap
[One paragraph restating the brief and any inferred context]

## Board verdicts at a glance
| Seat               | One-line position |
| Ethnographer       | ... |
| Storyteller        | ... |
| Channel Strategist | ... |
| Producer           | ... |
| Funnel Engineer    | ... |
| Community/PR Lead  | ... |
| Moonshot           | (reframing) |
| Contrarian         | ... |

## Where the board agrees
[Convergent points worth treating as decisions]

## Where the board disagrees
[Real tensions — name them, don't smooth them over. Each entry attributes the position to the seat(s) that took it.]

## The plan I'm proposing
[Synthesizer's recommended plan, addressing the strongest dissent explicitly. Single voice.]

## What I'm accepting as risk
[The Contrarian's strongest objection that the plan tolerates]

## Pre-launch checklist
[What needs to be true before spend / publication]

## First 30 days — concrete actions
1. ...
2. ...
3. ...

## Metrics to track
[KPIs the Funnel Engineer & Channel Strategist agree are the leading indicators of this plan working — typically 3-5 numbers, with the baseline (if known) and the target window]

## Open Questions

*Generated via deep analysis after synthesis — resolve before execution. Answer these in the saved memo file (tick the checkboxes), then run `/marketing-board --consolidate` to fold decisions into the memo (Workflow 4).*

| ID    | Question                       | Status |
| ----- | ------------------------------ | ------ |
| OQ-01 | [Decision blocking execution]  | Open   |
| OQ-02 | ...                            | Open   |

### OQ-01: [Question title]

**Question:** [Clear statement]

**Why it matters:** [What unblocks once resolved]

**Possible answers:**

- [ ] Option A
- [ ] Option B
- [ ] Option C

**Status:** Open
```

## Success Metrics

| Metric                                                          | Baseline | Target                      | Measurement                                                                                                                                                                                                                                                                |
| --------------------------------------------------------------- | -------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Memo covers all eight lenses with distinguishable contributions | N/A      | 100% of deliberations       | Author review of first 5 outputs                                                                                                                                                                                                                                           |
| Plan specific enough to act on without follow-up clarification  | N/A      | ≥3 of 5 first deliberations | Author qualitative judgment                                                                                                                                                                                                                                                |
| Reuse on a second SaaS without plugin modification              | N/A      | Works on next product       | Run on a non-empleo product post-launch                                                                                                                                                                                                                                    |
| Contrarian surfaces ≥1 risk the author hadn't considered        | N/A      | ≥3 of 5 deliberations       | Author qualitative judgment                                                                                                                                                                                                                                                |
| At least one "where the board disagrees" entry per memo         | N/A      | ≥80% of memos               | Author review                                                                                                                                                                                                                                                              |
| First empleo.digital acceptance test produces a usable plan     | N/A      | Pass on first run           | Author drafts the brief by having Claude read `~/proyectos/empleo/lp/empleo_lp` and propose a draft brief (per OQ-08); author edits and runs `/marketing-board`. Bootstrap of `agent_docs/marketing/` runs first (hard-fail per OQ-14) and is grounded in the same LP read |

## Scope

### In scope (v1)

- One slash command (default: `/marketing-board <brief>`)
- `--consolidate` flag for the slash command (memo OQ resolution — Workflow 4)
- Automatic memo persistence — every deliberation writes the memo to `marketing-plans/<product>-<YYYY-MM-DD>.md` in the working directory (per FR-20)
- Eight subagent definition files under `agents/`
- Synthesizer logic embedded in the slash command body (per NFR-04)
- **Bootstrap skill** (`/marketing-board:bootstrap`) — scaffolds `agent_docs/marketing/INTERVIEW.md` and consolidates into the six knowledge-base files (Workflow 5, FR-16 / FR-17 / FR-18 / FR-19)
- Six-file knowledge-base contract (`product.md`, `audience.md`, `competition.md`, `business.md`, `distribution.md`, `constraints.md`) read by agents at deliberation start
- Plugin README explaining seats, briefing tips, design rationale, and how to bootstrap
- Plugin metadata + marketplace registration
- Default tool allowlist for each agent (Read/Grep/Glob/WebSearch/WebFetch)
- Default model = `sonnet` per agent

### Out of scope (v1) — future child PRDs

- **Email sequence drafting skill** — PRD'd at [`01-email-sequence-skill.md`](01-email-sequence-skill.md); drafts the full content of the Funnel Engineer's email-sequence outlines into ESP-paste-ready Markdown with variant checkboxes. Implementation deferred to v2 of marketing-board
- **Video script skill** (`02-video-script-skill.md`) — scripts/shotlists for Gemini Omni and HeyGen Hyperframes
- **Landing-page audit skill** (`03-lp-audit-skill.md`) — conversion audit against the Funnel Engineer's plan
- **Configurable seat lineup** — let user swap in additional / alternative seats per deliberation (SEO Lead, Brand/Voice Director, Operator/Execution Realist), or omit specific seats
- Persisted vault integration (saving memos into an Obsidian vault)
- Multi-language synthesis controls (currently: output language follows brief language)

### Out of scope (forever)

- Campaign execution / message sending
- Performance tracking / attribution
- Multi-user team features

## Constraints & Dependencies

### Constraints

- Must follow this marketplace's conventions (see [CLAUDE.md][1]): version bumped on every change; supporting files load on demand; plugin owns its own discipline.
- Synthesizer is the load-bearing artifact — quality of v1 depends primarily on the synthesizer prompt in the slash command body.
- Agent count locked at eight for v1. Adding seats dilutes orthogonality; removing seats loses an angle.
- Must produce useful output on empleo.digital as the first acceptance test (see Success Metrics).
- Must use only documented Claude Code subagent mechanism (frontmatter fields per [the subagents docs](https://code.claude.com/docs/en/sub-agents)) — no custom orchestration.
- Knowledge-base files live in `agent_docs/marketing/` at the project root (not in the plugin directory, not in `~/.claude/`). Per-project context belongs to the project, so each SaaS owns its own files.
- Deliberation **requires** `agent_docs/marketing/` to be present with all six expected files. Bootstrap is **required** (not merely recommended) before the first deliberation on any new project — the slash command hard-fails otherwise with a single actionable prompt to bootstrap. (Per OQ-14 resolution.)
- `INTERVIEW.md` is the single source of truth during bootstrap-filling. Consolidation regenerates the six files; direct edits to those files after consolidation are encouraged for maintenance but require re-bootstrap only for major reframings.

### Dependencies

- **Claude Code subagent mechanism** — parallel dispatch via Agent tool, per-agent `model`/`tools`/`description` frontmatter. Per the docs: subagents cannot spawn other subagents (drives NFR-04).
- **`ceo-board` plugin** — pattern source; v1 architecture mirrors it.
- **Marketplace conventions** — `CLAUDE.md`, `README.md`, `marketplace.json` in this repo.

No external services, API keys, or paid dependencies for v1.

## Architecture Decisions (locked for v1, not implementation detail)

These are not implementation details (TDD territory) — they're scope decisions that affect what v1 *is*.

1. **Synthesizer is the slash command, not a separate agent.** Per the subagent docs, subagents can't spawn other subagents. Mirroring `ceo-board`, synthesis happens in the main session with the slash command's body as its system prompt extension. (NFR-04)
2. **Eight seats, not configurable in v1.** Configurability is v2 to avoid scope bloat. (Out of scope.)
3. **Read-only agents.** None of the eight can write to disk; their value is reports returned to the main session. (FR-09)
4. **No custom orchestration.** Plugin uses only Anthropic's documented subagent fields. (NFR-03)
5. **Memo structure is fixed.** Format defined in the slash command body so output stays predictable. (FR-05)
6. **Agents own the contract; the bootstrap skill owns the materialization.** Agents declare which `agent_docs/marketing/*.md` files they expect (file contract documented in [Product Knowledge Base](#product-knowledge-base-agent_docsmarketing)); the bootstrap skill is solely responsible for getting those files populated via `INTERVIEW.md`. This separation lets the agents be re-tuned without touching the bootstrap UX, and lets the interview UX evolve without breaking the agents. (FR-15, FR-16, FR-18)
7. **Single interview file, not per-file scaffolds.** Bootstrap creates one `INTERVIEW.md` (not six skeleton files) because (a) one place to fill is friendlier than navigating six, (b) the consolidator can cross-reference sections (e.g. extract anti-personas mentioned in audience into a callout in competition), (c) un-filled sections are visible at a glance in one document. (FR-17, FR-18)
8. **No `memory:` field on agents in v1.** Per the [Claude Code subagent docs](https://code.claude.com/docs/en/sub-agents), agents can opt into persistent memory at user/project/local scopes. v1 declines this — agents start fresh every deliberation. Cross-deliberation continuity comes from the `agent_docs/marketing/` knowledge base (file-based, user-editable, version-controllable), not from opaque subagent memory. Memory is easier to add later than to remove. (Resolves OQ-09.)
9. **Dangerous actions default to refusal; memo persistence is automatic.** `/marketing-board:bootstrap` on a project that already has `agent_docs/marketing/` aborts; full re-scaffolding requires explicit `--reset`. `--consolidate` is always safe to re-run (idempotent regeneration of target files). The deliberation memo is **always written to `marketing-plans/` automatically** — it is the durable artifact the user iterates on, and the answer-the-Open-Questions workflow depends on the file existing. A same-day re-run is suffixed, never an overwrite, so an annotated memo is never lost. (Resolves OQ-13, FR-19, FR-20 — reverses the earlier opt-in `--save` design; see OQ-04.)
10. **Single-voice plan, attributed disagreements.** The memo synthesizes a single recommendation (one voice) — except in *Where the board disagrees*, where each position is named with its seat(s). Attribution where it teaches; single voice where it acts. (Resolves OQ-10, reflected in memo template.)
11. **Hard-fail on missing knowledge base.** Deliberation requires all six `agent_docs/marketing/*.md` files. If any are absent, the slash command aborts at the frame step with a single actionable prompt to run `/marketing-board:bootstrap`. We chose blocking over silent degradation because brief-only output looks like an answer but is fundamentally ungrounded — the deliberation pattern depends on the knowledge base being authoritative. (Resolves OQ-14, FR-15.)

## Related Documents

- [`plugins/ceo-board/`](../../../plugins/ceo-board) — architectural precedent
- [`README.md`](../../../README.md) — marketplace overview
- [`CLAUDE.md`](../../../CLAUDE.md) — repository conventions for plugin authoring
- Claude Code subagents reference: https://code.claude.com/docs/en/sub-agents

**Child PRDs** (specs written ahead of implementation):

- [`01-email-sequence-skill.md`](01-email-sequence-skill.md) — email sequence drafting (consumes Marketing Plan Memo). Spec written 2026-05-22; implementation deferred to v2
- `02-video-script-skill.md` — Gemini Omni / Hyperframes scripting (placeholder)
- `03-lp-audit-skill.md` — landing page conversion audit (placeholder)
- `04-seat-configurability.md` — per-deliberation seat swap and additional seat options (SEO Lead, Brand/Voice Director, Operator, etc.) (placeholder)

## Open Questions

*All questions resolved and integrated as of 2026-05-20. Resolution log below kept for the decision trail; remove if/when this PRD is shared externally.*

### Resolution log

| ID    | Question (short)                   | Resolution                                                                                                                                                                                         | Integrated in                                      |
| ----- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| OQ-01 | Slash command name                 | `/marketing-board`                                                                                                                                                                                 | FR-01, Workflow 1                                  |
| OQ-02 | Memo structure                     | Accept v1 template **and** add a *Metrics to track* section                                                                                                                                        | Memo template                                      |
| OQ-03 | Brief input format                 | Hybrid — free-form input + recommended brief template in the README                                                                                                                                | FR-11                                              |
| OQ-04 | Output persistence                 | **Memo always saved automatically** to `marketing-plans/<product>-<YYYY-MM-DD>.md`; user answers Open Questions in that file. (Revised from the original optional-`--save`-default-off resolution) | FR-20, Scope, Architecture Decision 9              |
| OQ-05 | Web access                         | All eight seats get `WebSearch` + `WebFetch`                                                                                                                                                       | FR-09, The Eight Seats                             |
| OQ-06 | Model selection                    | **`opus`** for Storyteller, Moonshot, Contrarian (narrative subtlety, asymmetric reasoning, sharp skepticism); `sonnet` for the other five                                                         | NFR-05, The Eight Seats                            |
| OQ-07 | Moonshot + Community/PR Lead seats | Both added in v1 (8-seat lineup)                                                                                                                                                                   | The Eight Seats                                    |
| OQ-08 | Empleo.digital acceptance brief    | Claude reads LP at `~/proyectos/empleo/lp/empleo_lp` and proposes draft brief; author edits and runs                                                                                               | Success Metrics                                    |
| OQ-09 | Cross-deliberation agent memory    | No `memory:` field on agents in v1 — continuity comes from `agent_docs/marketing/`                                                                                                                 | Architecture Decision 8                            |
| OQ-10 | Synthesis transparency             | Attribute only in *Where the board disagrees*; single voice elsewhere                                                                                                                              | Memo template, Architecture Decision 10            |
| OQ-11 | Knowledge-base path                | **`agent_docs/marketing/`** (aligns with `bootstrap-vault` pattern from this marketplace)                                                                                                          | G-07, FR-15/16/18/19, Product Knowledge Base, etc. |
| OQ-12 | Bootstrap interview UX             | Template-first + embedded *"ask Claude to interview me"* hint per section                                                                                                                          | FR-17, Workflow 5                                  |
| OQ-13 | Bootstrap re-run behavior          | `bootstrap` aborts if files exist; `--reset` for explicit re-scaffold; `--consolidate` always idempotent                                                                                           | FR-19, Architecture Decision 9                     |
| OQ-14 | Missing-context surfacing          | **Hard-fail** — slash command aborts at frame step with prompt to run `/marketing-board:bootstrap`. No brief-only fallback                                                                         | FR-15, Architecture Decision 11, Workflow 1 step 3 |
| OQ-15 | INTERVIEW.md localization          | **Revised by TDD 02 OQ-B-06:** v1 ships English-only; the `--lang <code>` flag + runtime translation is deferred to v1.1 (original resolution was an explicit `--lang` flag in v1)                 | FR-16, Workflow 5                                  |

[1]: ../../../CLAUDE.md
