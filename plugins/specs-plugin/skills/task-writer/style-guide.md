# Task Style Guide

Conventions for task files, the board, and the archive index produced by the `task-writer` skill. SKILL.md is the overview; this file is the reference for *how* each piece is written.

## Contents

- Document structure and section order
- Frontmatter
- Title and numbering
- Lead paragraph
- Why (evidence discipline)
- Scope (Do NOT change, Out of scope)
- Constraints and Sequencing
- Done when (hybrid acceptance criteria)
- Verification
- Decisions (decided vs open)
- Next step
- Shipping record (Shipped, Decisions worth not re-litigating, Follow-ups)
- Size and priority scales
- The board
- The archive index
- Branch and commit
- Anti-patterns

## Document Structure and Section Order

1. YAML frontmatter
2. `# NNN — <title>` (the first line after the frontmatter, always)
3. Lead paragraph (no heading)
4. `## Shipped`, `## Decisions worth not re-litigating`, `## Follow-ups` — **archived files only**, in this position so the outcome is read before the history
5. `## Why`
6. `## Scope` (with `### Do NOT change` and `### Out of scope`)
7. `## Constraints` (only when a project rule bounds the approach)
8. `## Sequencing` (only when dependencies exist)
9. `## Done when`
10. `## Verification`
11. `## Decisions` (omit when there is nothing decided and nothing open)
12. `## Related` (only when there are links worth following)
13. `## Next step`
14. `## Notes` (reformat only: content that fit no section)

Task files do **not** use horizontal rules between sections; they are shorter than bug docs and the headings carry the structure.

## Frontmatter

YAML, six required keys, one optional, in this order:

```yaml
---
id: 009
title: CLOSING intent — let the agent end a conversation
status: todo
priority: P1
size: Medium
updated: 2026-09-03
note: needs a PRD first
---
```

| Key        | Holds                                                                                                  |
| ---------- | ------------------------------------------------------------------------------------------------------ |
| `id`       | The permanent three-digit id, matching the filename and the H1                                         |
| `title`    | Short, specific; quote it when it contains a colon (`title: 'Bug: …'` is a smell — bugs are not tasks) |
| `status`   | `todo` / `doing` / `blocked` / `shipped` — `shipped` only under `archived/`                            |
| `priority` | `P0`–`P3` (see scales below)                                                                           |
| `size`     | `Small` / `Medium` / `Large` (see scales below)                                                        |
| `updated`  | ISO date of the last substantive edit; bump it every time                                              |
| `note`     | Optional one-liner the board may surface: `needs a PRD first`, `waiting on vendor`                     |

Frontmatter is the structured truth; the board copies from it. Prose about the state of the task belongs in the lead paragraph, never in a frontmatter value.

## Title and Numbering

- H1: `# NNN — <title>` with an em dash, matching `title` in the frontmatter.
- `NNN`: zero-padded, sequential across `tasks/` and `archived/`, **permanent**. New tasks take the next free number; archived numbers are never reused.
- Filename: `NNN-kebab-slug.md`. The slug may be tidied later; the number may not.
- Backticked identifiers are welcome in the title when they are what the task is about, e.g. a title of `Silent pivot turns in manager_v2` with the module name backticked.

## Lead Paragraph

One paragraph directly under the H1. It answers "where does this stand and what shape is the work?" so the reader can decide whether to open the rest.

> Partly shipped: `GenerateInfoAnswer` got history on 2026-09-03 and is deployed. The remainder is smaller than it first looked — see Remaining. The size above is for that remainder.

> The work is provider-side; the code and the deploy already handle a number change.

As the task evolves, this paragraph is what changes most. Keep it current.

## Why (evidence discipline)

The reason the task exists, written as **dated evidence** so the next session does not re-investigate.

✅ "Generation signatures do not receive history: `grep -c history ai/generation/signatures.py` returns 0, while all six extraction signatures take a `history` input (`ai/extraction/signatures.py:128, 347, 540, 634, 746, 866`). Checked 2026-09-03."

❌ "The generator doesn't see the conversation history."

Rules:

- Every factual claim carries a date and a `file:line`, a command with its output, or an observation in a named environment.
- Tables are welcome when comparing layers, environments, or before/after.
- Quote the project's own docs or rules when they are the reason (a `CLAUDE.md` rule, an architecture doc).
- When the Why grew out of a bug, say so and link the bug; the task is the improvement, the bug stays in `specs/bugs/`.

## Scope (Do NOT change, Out of scope)

Numbered steps saying **WHAT** changes: file, function, setting, playbook. Signatures or snippets no longer than 5 lines; the diff belongs in the PR.

```markdown
## Scope

1. Add `CLOSING` to `IntentType` (`conversation/session.py:20-38`).
2. Let the `ExtractIn*` signatures emit `intent_switch_to: closing`.
3. Map that to `EndSession(EndSessionReason.CUSTOMER_REQUEST)`, clearing `active_intent`.

### Do NOT change

- The sticky-intent dispatch in `manager_v2.py:590-593` — it is documented behavior, not the defect.

### Out of scope

- Re-enabling `GenerateCorrectionAcknowledgment` (`enabled=False`); separate task.
```

Both guardrail subsections are mandatory. An empty `Out of scope` is a sign the investigation stopped early — there is always tempting nearby work.

## Constraints and Sequencing

`## Constraints` holds project rules that bound *how* the task may be done: a `CLAUDE.md` rule, an invariant pinned by a named test, a compliance limit, a component library preference. Quote the rule and cite where it lives.

`## Sequencing` holds dependencies in both directions and the reason for the ordering:

> **Wait for evidence before extending the pattern.** The first change is a behavioral bet deployed on 2026-09-03 and not yet observed in real traffic. Do `present_alternatives` after that; treat `acknowledge_capture` as optional given how rarely its gate opens. Sequenced after task 002 — decide Loki vs MLflow first.

A `blocked` task names its blocker here.

## Done when (hybrid acceptance criteria)

Under one `## Done when` heading, provide **both** a table and a mirrored checklist, exactly as bug docs do, so `ac-checker` can parse it:

```markdown
## Done when

| #    | Context                          | Action                     | Expected result                                          |
| ---- | -------------------------------- | -------------------------- | -------------------------------------------------------- |
| AC-1 | Session on INFO, user says "gracias" | Turn is processed      | Session ends with `reason: customer_request`             |
| AC-2 | Session mid-booking, user says "gracias" | Turn is processed  | Session stays on BOOKING; the thanks is acknowledged     |

### Checklist

- [ ] AC-1: A closing on INFO ends the session with `customer_request`
- [ ] AC-2: A closing mid-booking does not end the session
```

Every criterion is observable — a rendered value, a log line, a row, a command's output. "Works correctly" is not a criterion. When a task is shipped, each box is ticked with a short pointer to its evidence.

## Verification

Three parts, each present when applicable:

1. **Tests to add or run** — unit for changed logic, E2E for a user-facing flow, one unchanged-behavior test per `Do NOT change` item. Name the files or corpora.
2. **Confirm in the running system** — the command, URL, dashboard panel, or host check that proves the outcome live. For operational tasks this is the whole verification: `bin/celp whatsapp` returning the verified number, a deploy marker in the logs, a DNS answer.
3. **Evidence on ship** — dated output pasted or referenced.

## Decisions (decided vs open)

One `## Decisions` section, two kinds of entry:

- **Decided** (decide-and-record): the question struck through, then the answer in bold with its date, then the rationale.

    ```markdown
    - ~~Should a closing mid-booking end the session?~~ **Decided 2026-09-03: no.** `CLOSING` is reachable only from `INFO`, `GREETING` and `FALLBACK`. A user saying "gracias" halfway through slot collection means "thanks for that answer". Context decides, never the word.
    ```

- **Open** (ask-tier only): an `OD-NN` id, the fork, the choices with consequences, and a recommendation.

    ```markdown
    - **OD-01 (open):** Should the validation reject a zero-stage service outright? — A) reject on create/update: dead data becomes impossible, existing rows need a migration check; B) allow and render `0`: no migration, the booking handler must keep guarding. **Recommended: A** — the handler already cannot book it.
    ```

Omit the section when there is nothing decided and nothing open. Never pad it.

## Next step

Mandatory, one to three sentences, a single concrete action: "Fix the two calculations, add the validation, add the regression test asserting a stage-less service renders `0`." On a PRD stub it is "Write the PRD" plus what to study first.

## Shipping Record

Added at archive time, placed right after the lead paragraph so the reader meets the outcome before the history:

- `## Shipped` — what landed (a table of piece -> where is good), the commits or PR, dated verification evidence, and the actual size versus the estimate ("Size was Medium, as estimated").
- `## Decisions worth not re-litigating` — only for choices that deliberately depart from the obvious or documented way. Each says *why*, and where it is pinned (a test, a code comment).
- `## Follow-ups` — new work the task revealed. Each is **filed as its own task** with a link (`-> task 015`). Something that must be done for *this* task to count as done is a leftover, not a follow-up; finish it or split the task.

## Size and Priority Scales

**Size** estimates build effort only. It says nothing about priority and nothing about how long the decisions inside will take.

| Size       | Meaning                                                                                   |
| ---------- | ----------------------------------------------------------------------------------------- |
| **Small**  | A focused change, roughly a day or less, no new subsystem                                 |
| **Medium** | Several files or both stacks, or something operational that must be deployed and verified |
| **Large**  | Needs a PRD first — the task is a stub whose next step is the PRD                         |

**Priority** is about consequence, independent of size.

| Priority | Meaning                                                         |
| -------- | --------------------------------------------------------------- |
| **P0**   | The product is broken or the pilot/launch is at risk; do it now |
| **P1**   | Next up; the product is worse every day this waits              |
| **P2**   | Worthwhile; pick it up when P1 is clear                         |
| **P3**   | Someday; kept so the reasoning is not lost                      |

## The Board

`specs/backlog/BACKLOG.md`. Two sections, one table each, five columns:

```markdown
## Doing

| #   | Task                  | Priority | Size | Hook |
| --- | --------------------- | -------- | ---- | ---- |
| —   | *(nothing in flight)* |          |      |      |

## Next

| #                                        | Task                        | Priority | Size  | Hook                                                  |
| ---------------------------------------- | --------------------------- | -------- | ----- | ----------------------------------------------------- |
| [008](tasks/008-whatsapp-production-number.md) | WhatsApp production number | P1 | Small | The pilot is on a sandbox number capped at 5 recipients |
```

- **The board is the index; the task file is the truth.** Only id, title, priority, size and a one-line hook live here.
- **Status is the section.** Doing or Next; shipped rows leave. A `blocked` task stays under Next with a hook beginning `Blocked —`.
- **Order**: by priority, then by size ascending, so the top of Next is the best thing to start.
- **The hook** says why it matters or what unlocks it — the thing that makes someone open the file.
- The board ends with a `## How this works` section (see the template). A project may extend it; the skill follows what it finds there.

## The Archive Index

`specs/backlog/archived/README.md`. One row per shipped task, added at archive time:

```markdown
| #                                   | Task                          | Shipped    | What it settled                                                              |
| ----------------------------------- | ----------------------------- | ---------- | ---------------------------------------------------------------------------- |
| [001](001-sentry-error-tracking.md) | Sentry error tracking         | 2026-09-03 | Two projects, not one; no `PlugCapture` on Bandit; PII scrubbed by structure |
```

The last column says what the task **settled** — which questions are now closed — not what it did. The commit history already records what it did; a future reader needs to know which choices not to reopen.

## Branch and Commit

For `--do` runs, work lands on `task/<NNN>-<slug>` so a task never commits straight to a protected branch. Auto-created when on `main`/`master`/`develop`/`release/*`; reused when already on a feature branch; `--branch NAME` and `--no-branch` override.

**The skill never commits.** On ship it stages the code, the tests, the `git mv` into `archived/`, the board edit, the index row, and any follow-up task files — together — and hands off. The user reviews the staged diff and commits; the move rides in that commit.

**Pre-ship gate**: nothing is archived or advised for commit until the five ship gates hold — evidence-backed Why, bounded scope, done criteria met, verified in the running system, settled and handed off.

## Anti-Patterns

| Anti-pattern                          | Why it fails                                                                       |
| ------------------------------------- | ---------------------------------------------------------------------------------- |
| A bug filed as a task                 | Skips reproduction, root cause and the red->green test; the defect ships again     |
| Undated evidence                      | Rots silently; the next session re-investigates or, worse, trusts a stale claim    |
| Scope without `Do NOT change`         | Invites "while I'm here" changes that widen the blast radius                       |
| "Works correctly" as a done criterion | Cannot be checked, so the task is never really done                                |
| Shipped on green tests alone          | A deploy can succeed on a stale commit; only the running system proves the outcome |
| Leftovers labelled follow-ups         | Hides unfinished work behind a tidy heading                                        |
| Archive row says what it did          | The commit log has that; the reader needs which questions are closed               |
| Renumbering or reusing ids            | Breaks every citation in commits, STATUS files, and runbooks                       |
| Manufactured open decisions           | Stalls ready work on a review nobody needed                                        |
| Status prose in frontmatter           | Breaks the board's copy; prose belongs in the lead paragraph                       |
