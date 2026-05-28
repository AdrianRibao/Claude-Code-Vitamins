# marketing-board

An eight-seat marketing deliberation board for SaaS launches, pointed at one question: *how do we actually take this product to market?*

Install the plugin, bootstrap a per-project knowledge base once, then call `/marketing-board <brief>` to convene eight specialist marketers in parallel and synthesize a Marketing Plan Memo.

## The board

Eight specialist subagents, each with isolated context and a sharp lens:

| Agent                  | Lens                                                                |
| ---------------------- | ------------------------------------------------------------------- |
| **Ethnographer**       | ICP, JTBD, personas, attention map, vocabulary of the buyer         |
| **Storyteller**        | Positioning, message house, hero narrative, taglines (opus)         |
| **Channel Strategist** | Channel mix, budget allocation, sequencing, CAC math                |
| **Producer**           | Editorial calendar, asset list, production briefs, distribution map |
| **Funnel Engineer**    | Funnel map, email sequences, A/B priorities, retention loops        |
| **Community/PR Lead**  | Communities, ambassadors, launch-day sequence, PR pitch angles      |
| **Moonshot**           | 10× reframing, asymmetric bets, unlock moves (opus)                 |
| **Contrarian**         | Hidden assumptions, base rates, failure modes (opus)                |

Plus a `/marketing-board` slash command that fans out all eight in parallel and synthesizes a Marketing Plan Memo. And a `:bootstrap` skill that scaffolds the product knowledge base each board member reads before deliberating.

## Install

```
/plugin marketplace add AdrianRibao/Claude-Code-Vitamins
/plugin install marketing-board@claude-code-vitamins
```

## How to use

The plugin has **two commands** you call in this order. Step 1 is required before Step 2 will run — `/marketing-board` hard-fails if the knowledge base isn't there yet.

### Step 1 — Bootstrap the product knowledge base (one-time per SaaS project)

Without this, the board has no product context to deliberate against.

```
/marketing-board:bootstrap
```

This writes a single file: `agent_docs/marketing/INTERVIEW.md` — a guided interview with ~36 questions across **product, audience, competition, business, distribution, constraints**, each with examples and checkbox choices where helpful.

Fill it in. You have three options:

- Paste from existing docs (pitch deck, landing page, ICP doc) under each section.
- Ask Claude to interview you section-by-section: *"Walk me through `audience.md` questions one at a time."*
- Do it solo, by hand.

When the interview is complete, consolidate:

```
/marketing-board:bootstrap --consolidate
```

This parses `INTERVIEW.md` and writes the six knowledge-base files the board reads:

```
agent_docs/marketing/
├── product.md          # what the product is and does
├── audience.md         # ICP, JTBD, personas, vocabulary
├── competition.md      # competitors, alternatives, category
├── business.md         # pricing, unit economics, goals
├── distribution.md     # channels you have / want / have tried
└── constraints.md      # budget, timeline, team, hard nos
```

You can edit these files directly afterwards — the board reads the final state, not the interview.

### Step 2 — Convene the board on a brief (every deliberation)

```
/marketing-board We launch in 6 weeks. Spanish-only blue-collar job board.
                 Budget €5k. Founder is solo. Goal: 500 verified candidates
                 and 20 paying SMBs in the first 60 days. What's the plan?
```

What happens:

1. The command verifies the knowledge base exists (hard-fails with `Bootstrap your product context first: /marketing-board:bootstrap` otherwise).
2. All **8 specialists run in parallel**, each reading only the knowledge-base files relevant to their lens.
3. The main session synthesizes their reports into a **Marketing Plan Memo**: verdicts-at-a-glance, where they agree / disagree (attributed), a single-voice plan, accepted risks, pre-launch checklist, first-30-days actions, metrics, and Open Questions.
4. The memo is **saved automatically** to `marketing-plans/<product-slug>-<YYYY-MM-DD>.md`. No flag needed.

### Step 3 — Iterate on the memo (as many times as you want)

Open the saved memo file. Answer the Open Questions in place: tick the `- [ ]` checkboxes, add notes inline. Then:

```
/marketing-board --consolidate
```

The command reads the same file, folds your answers into the relevant sections, marks resolved questions, tightens prose, and **rewrites the same file in place**. Repeat as you go.

## Knowledge base — what the board reads

Each board member loads only the files relevant to their lens, so context stays sharp:

| File              | Read by                                                                                |
| ----------------- | -------------------------------------------------------------------------------------- |
| `product.md`      | All seats                                                                              |
| `audience.md`     | Ethnographer, Storyteller, Channel Strategist, Producer, Community/PR Lead, Contrarian |
| `competition.md`  | Storyteller, Moonshot, Contrarian                                                      |
| `business.md`     | Channel Strategist, Funnel Engineer, Contrarian                                        |
| `distribution.md` | Channel Strategist, Producer, Community/PR Lead, Contrarian                            |
| `constraints.md`  | Moonshot, Contrarian                                                                   |

Each file has a required `##` section structure (enforced by `/marketing-board`'s startup check). You're free to add extra `##` sections — they're preserved across `:bootstrap --consolidate` re-runs and visible to all agents.

## How it works

1. `/marketing-board <brief>` verifies the knowledge base exists. If any file or required section is missing, it hard-fails with `"Bootstrap your product context first: /marketing-board:bootstrap"`.
2. The main Claude Code session frames the brief, then dispatches all 8 board members **in parallel** via the Agent tool. Each subagent runs in its own fresh context, loads only its required knowledge-base files, and returns a structured report.
3. The main session reads the synthesizer prompt from `reference/synthesizer-prompt.md` and produces a single Marketing Plan Memo — plus an Open Questions section generated by deep analysis (same shape as the prd-writer / tdd-writer skills produce in their Phase 2). The memo is written to `marketing-plans/<product>-<YYYY-MM-DD>.md` automatically.
4. Answer the Open Questions by editing the saved memo file (tick the checkboxes), then run `/marketing-board --consolidate` to fold the answers in — it rewrites the same file in place.

Synthesis happens in the main session so the final memo lives in the same context the user is working in.

## Flags

| Flag            | Purpose                                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------------------- |
| *(none)*        | Full deliberation: dispatch all 8, synthesize memo + OQs, save the memo to `marketing-plans/` automatically         |
| `--consolidate` | Read the saved memo file, integrate the OQ answers you marked in it, mark resolved, tighten prose, rewrite in place |

Bootstrap flags:

| Flag            | Purpose                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------- |
| *(none)*        | Scaffold `agent_docs/marketing/INTERVIEW.md`. Aborts if knowledge base already exists       |
| `--reset`       | Overwrite INTERVIEW.md only (does NOT touch the six knowledge-base files)                   |
| `--consolidate` | Parse INTERVIEW.md → write/overwrite the six target files. Idempotent for structured fields |
| `--lang <code>` | Reserved for v1.1 — currently rejected. Bootstrap ships English-only in v1                  |

## Tips for getting useful memos

- **Bootstrap properly.** A board reading a half-empty knowledge base will produce a half-empty plan. The interview takes 30-60 minutes; the payoff is every memo afterwards.
- **Give the board a sharp brief.** Constraints (budget, timeline, channels you've already tried) matter more than aspirations.
- **Re-run with a counter-brief.** After the memo, ask the board to deliberate on the *opposite framing*. The asymmetry is where the real plan lives.
- **Don't outsource the call.** The board sharpens your thinking; the judgment is yours.

## Customizing

Add a seat by dropping a markdown file in `agents/` with the standard frontmatter and the 5 required body sections (`## Identity`, `## Your lens`, `## How you think`, `## Context loading`, `## Output format`). Update `commands/marketing-board.md` to invoke it by name. Keep the description tight — that's what Claude Code uses for auto-routing.
