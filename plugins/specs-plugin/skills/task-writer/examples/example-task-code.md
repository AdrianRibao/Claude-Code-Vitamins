---
id: 009
title: CLOSING intent — let the agent end a conversation
status: todo
priority: P1
size: Medium
updated: 2026-09-03
---

# 009 — CLOSING intent — let the agent end a conversation

Touches the state machine, six extraction signatures and the golden corpora. The visible symptom is already patched (a bare "gracias" is now acknowledged), so this task is about the correctness of the session lifecycle, not a broken reply.

## Why

On 2026-09-03 a pilot conversation went:

```
user: Gracias
bot:  No tengo esa información todavía.
```

The reply was patched (commit `792e7ce` added a courtesy branch to `GenerateInfoAnswer`), but the underlying gap is untouched: **the agent has no way to end a conversation.** After "Gracias" the session stays on `active_intent: INFO` indefinitely, and the only exit is the 12-hour session window expiring.

The exit was designed and never wired. `python/src/celp_dspy/conversation/handler_result.py:55-56` defines `CUSTOMER_REQUEST = "customer_request"` with the docstring *User explicitly ended the chat ("gracias", "listo", "bye")*. `grep -rn CUSTOMER_REQUEST python/src/` (2026-09-03) returns the enum and an observability mapping — nothing emits it.

Three structural facts, all deliberate, explain why a handler-level fix cannot work:

1. **Intent is sticky.** `manager_v2.py:590-593` re-classifies only when `active_intent` is `None` or `GREETING` — the documented dispatch loop in `docs/architecture/pipeline.md`, not a bug.
2. **In-intent extractors discard the model's own verdict.** `ai/extraction/state_aware.py:668-669` keeps only `intent_switch_to` and forces the current intent otherwise.
3. **The architecture models pivots, not endings.** `Transfer` / `ReturnAfter` move between tasks; nothing expresses "the user is done" except `EndSession`, which only handlers emit on completion or error.

So a closing turn is neither a task nor a pivot, and falls through to "answer it as a question".

## Scope

1. Add `CLOSING` to `IntentType` (`conversation/session.py:20-38`).
2. Let the `ExtractIn*` signatures emit `intent_switch_to: closing` — the only channel that can move a sticky session.
3. Map that to `EndSession(EndSessionReason.CUSTOMER_REQUEST)`, clearing `active_intent`.
4. A TEMPLATE goodbye case in `ai/generation/registry.py` plus the gettext catalogue entry.
5. Golden corpus rows for closings across intents (`gracias`, `listo`, `nada más`, `hasta luego`), including the mid-booking case where a closing must **not** fire.
6. Confirm `SessionTerminationEvent` renders `reason: customer_request` — the mapping exists in `observability/termination.py:71`, so the dashboards should light up for free.

### Do NOT change

- The sticky-intent dispatch in `manager_v2.py:590-593`.
- The courtesy branch from `792e7ce` — it stays as the fallback for a thanks that is not a closing.
- Slot collection inside BOOKING; a closing must never abort a booking in progress.

### Out of scope

- Re-enabling `GenerateCorrectionAcknowledgment` (`enabled=False` in the registry).
- Tuning the 12-hour idle window.

## Constraints

- `CLAUDE.md`: *"For AI, never use keyword/pattern matching in conversation flow for detecting intentions."* An `if "gracias" in message` check is out. This has to be signature-level, which is also where the gap is.
- Inline Spanish literals are blocked by the pre-commit hook; the goodbye goes through the gettext catalogue.

## Done when

| #    | Context                                      | Action            | Expected result                                                               |
| ---- | -------------------------------------------- | ----------------- | ----------------------------------------------------------------------------- |
| AC-1 | Session on INFO, user says "gracias"         | Turn is processed | Session ends; `SessionTerminationEvent.reason == customer_request`            |
| AC-2 | Session on GREETING, user says "hasta luego" | Turn is processed | Same as AC-1                                                                  |
| AC-3 | Session mid-booking, user says "gracias"     | Turn is processed | Session stays on BOOKING; the thanks is acknowledged; no termination          |
| AC-4 | Session on INFO, user asks a question        | Turn is processed | Unchanged: answered on INFO (no false closings on the info corpus)            |
| AC-5 | Any closing                                  | Grafana panel     | "Session terminations by reason" shows `customer_request`, not `idle_timeout` |

### Checklist

- [ ] AC-1: A closing on INFO ends the session with `customer_request`
- [ ] AC-2: A closing on GREETING ends the session with `customer_request`
- [ ] AC-3: A closing mid-booking does not end the session
- [ ] AC-4: The info corpus produces zero closings
- [ ] AC-5: The terminations panel distinguishes `customer_request` from `idle_timeout`

## Verification

- **Tests**: corpus rows in `ExtractInInfo.yaml`, `ExtractInGreeting.yaml`, `ExtractInBooking.yaml` (AC-1..3); the full info corpus as the unchanged-behavior test for AC-4; a unit test on the `intent_switch_to: closing` -> `EndSession` mapping.
- **Confirm live**: after deploy, send "gracias" from a pilot phone on an INFO session and check the terminations panel and the `session_terminated` log line (AC-5).

## Decisions

### D-01 — Should a closing mid-booking end the session?

**Decided 2026-09-03.** No.

**Why.** `CLOSING` is reachable only from `INFO`, `GREETING` and `FALLBACK` — the same shape as `GUEST_ALLOWED_INTENTS`. A user saying "gracias" halfway through slot collection means "thanks for that answer". Context decides, never the word.

### D-02 — Does ending differ from lapsing?

**Decided 2026-09-03.** Yes, and it is the whole point.

**Why.** Both leave the user without a session; only the recorded reason differs. Today a happy ending and an abandonment are filed identically (`idle_timeout` / `abandoned`), so the completion-rate panel cannot tell them apart.

## Related

- The golden corpus previously encoded this gap as expected behavior — `ExtractInInfo.yaml` had contradictory rows for "vale gracias", fixed in `792e7ce`. Re-read it before adding rows.
- Task 012 — silent pivot turns (shares the `intent_switch_to` path).

## Next step

Add `CLOSING` to `IntentType` and write the three corpus rows first, so the extractor change is driven by failing rows.
