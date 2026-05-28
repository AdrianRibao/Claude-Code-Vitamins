---
description: Convene 8 marketing specialists on a SaaS launch brief; synthesize a Marketing Plan Memo. Requires `agent_docs/marketing/` (run `:bootstrap` first).
argument-hint: <launch brief | --consolidate [@<memo-path>]>
---

You are convening the **Marketing Board**. Eight specialist subagents will deliberate in parallel and you will synthesize their reports into a single Marketing Plan Memo.

## The arguments

$ARGUMENTS

## Procedure

Execute these steps **in order**. Do not skip ahead.

### Step 1 — Verify the knowledge base

Check that `agent_docs/marketing/` exists in the working directory and contains all six required files:

- `product.md`
- `audience.md`
- `competition.md`
- `business.md`
- `distribution.md`
- `constraints.md`

Verify the knowledge base with two batched calls — do not read full file bodies or use shell `ls`:

1. One `Glob` for `agent_docs/marketing/*.md` — confirms which of the six files exist.
2. One `Grep` for `^## ` scoped to `agent_docs/marketing/` — returns every `##` header across all files in a single pass, which feeds the section-presence check below.

**If the directory or any required file is missing**, abort immediately with this exact message and stop:

```
Bootstrap your product context first: /marketing-board:bootstrap
```

**If a required file exists but a required `##` section is missing** (refer to the section contracts in the table further down this step), abort with:

```
Knowledge base file <file>.md is missing required section: <section>. Re-run /marketing-board:bootstrap --consolidate.
```

The required `##` headers per file:

| File              | Required `##` sections                                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `product.md`      | `Product description`, `Category`, `Value propositions`, `Positioning statement`, `Brand voice & tone`                 |
| `audience.md`     | `ICP`, `Personas`, `Anti-personas`, `Attention map`, `Vocabulary`, `Triggers to buy`                                   |
| `competition.md`  | `Alternatives`, `How we differ`, `Category conventions to inherit`, `Category conventions to break`                    |
| `business.md`     | `Pricing tiers`, `Funnel state`, `Budget envelope`, `Timeline`, `Geographic focus`                                     |
| `distribution.md` | `Owned channels`, `Content inventory`, `Founder communities`, `Aspirational influencers`, `Past PR`                    |
| `constraints.md`  | `Legal / compliance`, `Founder risk tolerance`, `Past failed experiments`, `Unfair advantages`, `Regulatory landmines` |

No partial-context fallback. No silent degradation.

### Step 2 — Parse flags

Scan `$ARGUMENTS` for the `--consolidate` flag:

- `--consolidate [@memo-path]` → jump to the **Consolidate sub-flow** below.
- *(no flag)* → continue with the main flow. The memo is saved to disk automatically at step 6 — there is no save flag.

`--consolidate` and the brief are mutually exclusive: `--consolidate` operates on an existing memo file, not a new brief.

### Step 3 — Frame the decision

Restate the brief in one paragraph. If the brief references project files (e.g. "look at the current LP", "factor in the homepage copy"), use Read/Grep/Glob to ground the framing in the actual artifacts before dispatching the board. Identify the implicit question the founder is really asking.

### Step 4 — Dispatch the board in parallel

Dispatch all 8 board members **concurrently in a single Agent-tool fan-out** (not 8 sequential calls). Each gets the framed brief plus any repo-aware context you gathered in step 3. Invoke by name:

- `ethnographer`
- `storyteller`
- `channel-strategist`
- `producer`
- `funnel-engineer`
- `community-pr-lead`
- `moonshot`
- `contrarian`

Each subagent will load its own knowledge-base files (per its `## Context loading` section) and return a structured report opening with `## <Seat> Report`.

**On agent failure (timeout or error from any seat)**: retry that seat **once**. If the retry succeeds, proceed with all 8 reports. If the retry also fails, proceed with the remaining N−1 reports and remember to:

- Render that seat's row in the verdicts table as `[FAILED: <reason>]`.
- Note the failure once in the memo's `## Brief recap`.
- Add a `[verify retry]` callout to the Open Questions section flagging the partial deliberation.

### Step 5 — Synthesize the Marketing Plan Memo + Open Questions

Read the synthesizer prompt from `${CLAUDE_PLUGIN_ROOT}/reference/synthesizer-prompt.md` and follow its instructions. The synthesizer prompt is authoritative for memo structure, the attribution convention, and the Open Questions generation contract.

Emit the memo **and** its Open Questions section in the same Claude turn — no separate sub-prompt.

### Step 6 — Save the memo

Every deliberation is saved automatically — there is no flag and no opt-out.

1. Derive a product slug from the **product name** — the proper-noun subject of `agent_docs/marketing/product.md`'s `## Product description` (e.g. a description that opens "empleo.digital is a Spanish-language job board…" → product name `empleo.digital` → slug `empleo-digital`). Kebab-case it: lowercase; spaces, dots and underscores become hyphens; strip other punctuation. If no clear product name can be identified, fall back to the working directory's basename, kebab-cased.
2. Create the `marketing-plans/` directory in the working directory if it doesn't exist.
3. Write the full memo (including the Open Questions section) to `marketing-plans/<product-slug>-<YYYY-MM-DD>.md`.
4. If a file at that path already exists (a same-day second deliberation), append `-<HHMMSS>` to the filename before `.md` — **never overwrite** an existing memo file.
5. Tell the user the saved path, and that they answer the Open Questions by editing that file — ticking the `- [ ]` checkboxes, adding notes — then running `/marketing-board --consolidate`.

## Consolidate sub-flow

Triggered when `$ARGUMENTS` contains `--consolidate`.

1. **Locate the memo file.**

    - If `--consolidate @<path>` was given, read that file.
    - Otherwise, find the **most recently modified `.md` file in `marketing-plans/`** and read it.
    - If no memo file is found, abort with: `No memo found to consolidate. Run /marketing-board <brief> first.`

2. **Parse the OQ section.** Identify the summary table and the per-OQ detail blocks. The detail-block shape is the locked Open Questions generation contract in `${CLAUDE_PLUGIN_ROOT}/reference/synthesizer-prompt.md` — parse against that; do not assume a different shape.

3. **Read the answers the user marked in the file.** A question is answered when:

    - one of its `- [ ]` options is ticked `- [x]`, and/or
    - the user added a note under the question.

   The file is the source of truth — do not scan the conversation for answers.

4. **For each answered OQ:**

    - Integrate the answer into the relevant memo section (plan, metrics, risks, checklist — whichever the OQ targets).
    - In the OQ detail block, mark `**Status:** Resolved (YYYY-MM-DD) — <1-line summary>`.
    - Update the summary table's `Status` column to `Resolved`.

5. **If all OQs resolve**, collapse the section to a brief resolution log.

6. **Write the consolidated memo back to the same file**, in place. Idempotent: running this twice on the same file + same in-file answers must produce a byte-identical file.

7. Do not re-dispatch the board. Tell the user which file was updated.

## Stay honest

If the board's input reveals the brief itself was wrong, say so in the memo. The point isn't to ratify the original framing — it's to make a *better launch*. The Contrarian's reluctant yes is more useful than the Storyteller's confident maybe.

Begin.
