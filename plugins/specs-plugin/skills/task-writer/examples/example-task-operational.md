---
id: 008
title: WhatsApp production number (off the sandbox)
status: todo
priority: P1
size: Small
updated: 2026-09-04
---

# 008 — WhatsApp production number (off the sandbox)

The work is provider-side; the code and the deploy already handle a number change. One prerequisite (an extra subscribed app) surfaced during the investigation and is sequenced first.

## Why

The pilot runs on a Meta **sandbox test number** (`verified_name: "Test Number"`, `code_verification_status: NOT_VERIFIED`, as returned by `GET /{phone_number_id}` on 2026-09-03). Test numbers are capped at **5 pre-registered recipients**, so the agent cannot reach the partner's actual clients. Discovered on 2026-09-03 while replacing the Meta app — the previous number was a sandbox one too, so the pilot has never had a real line.

Two facts from the same investigation shape the scope:

| Fact                                                                                | Where verified                                                                 | Consequence                                                                                               |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| Inbound fails closed when the id does not match `partners.whatsapp_phone_number_id` | `lib/celp/whatsapp/inbound.ex:41` (`WHATSAPP_UNMATCHED_PHONE_NUMBER_ID_ALERT`) | The id must change in the vault **and** the partner row, together                                         |
| Meta's `WA DevX Webhook Events 1P App` is still subscribed to the WABA              | `GET /{waba_id}/subscribed_apps`, 2026-09-04                                   | Every event arrives twice; the duplicate is rejected as `signature_mismatch` (5 in the 24h to 2026-09-04) |

## Scope

1. Unsubscribe the stray first-party app: `DELETE /{waba_id}/subscribed_apps` with a token for that app id. Confirm `signature_mismatch` rejections stop.
2. Add a real, verified phone number to the WABA (SMS/voice verification -> `code_verification_status: VERIFIED`).
3. Put the new `phone_number_id` into `service_env.WHATSAPP_PHONE_NUMBER_ID` (vault) **and** `partners.whatsapp_phone_number_id` (partner row) in the same change.
4. `ansible-playbook app.yml`, then `bin/celp whatsapp` to confirm the host sees the verified number.

### Do NOT change

- The fail-closed behavior on an unmatched `phone_number_id` — it is the guard that makes step 3 safe.
- The webhook signature check; `signature_mismatch` must keep rejecting.

### Out of scope

- A `signature_mismatch > 0` alert rule. It would page permanently while the stray app is subscribed; add it as a follow-up once step 1 has been confirmed for a full day.
- Multi-number routing (task 003).

## Constraints

- The runbook [`specs/runbooks/whatsapp-app-setup.md`](../../runbooks/whatsapp-app-setup.md) is the sequence of record; this task follows it and updates it where reality differed.
- Production credentials live only in the Ansible vault; nothing is pasted into the task file.

## Sequencing

Step 1 before step 2: with two apps subscribed the logs are ambiguous, and the verification of the new number would be read through duplicated events. Task 003 (multi-partner routing) builds on the single verified number this task produces.

## Done when

| #    | Context                                  | Action                          | Expected result                                                        |
| ---- | ---------------------------------------- | ------------------------------- | ---------------------------------------------------------------------- |
| AC-1 | Stray app unsubscribed                   | Tail inbound logs for 24h       | Zero `signature_mismatch` rejections                                   |
| AC-2 | New number added to the WABA             | `GET /{phone_number_id}`        | `code_verification_status: VERIFIED`, real `verified_name`             |
| AC-3 | Vault + partner row updated, deployed    | `bin/celp whatsapp` on the host | Prints the new id and `VERIFIED`; no `UNMATCHED_PHONE_NUMBER_ID` alert |
| AC-4 | A number not in the old 5-recipient list | Sends a message to the new line | The agent replies                                                      |

### Checklist

- [ ] AC-1: No `signature_mismatch` rejections in the 24h after unsubscribing the stray app
- [ ] AC-2: The new number reports `VERIFIED`
- [ ] AC-3: The host sees the new id; no unmatched-id alert after deploy
- [ ] AC-4: An unregistered recipient gets a reply

## Verification

- **Tests**: none to add — no code changes. The existing `inbound_test.exs` unmatched-id case covers the guard we rely on.
- **Confirm live**: AC-1 via Loki (`event_kind="whatsapp_inbound_rejected"` over 24h); AC-2 via the Graph API call; AC-3 via `bin/celp whatsapp`; AC-4 by messaging the line from a phone that was never on the sandbox list.

## Decisions

- ~~Add the `signature_mismatch` alert now?~~ **Decided 2026-09-04: no — after step 1 is confirmed.** It fires today for a benign reason, so a `> 0` rule would page immediately and permanently, and an alert that is wrong on day one teaches people to ignore the channel.
- ~~Change the vault first, then the partner row?~~ **Decided 2026-09-04: same change, same deploy.** The two must match or inbound fails closed; a split deploy guarantees a window where it does.

## Related

- Runbook: `specs/runbooks/whatsapp-app-setup.md`
- Task 003 — multi-phone -> multi-partner routing (builds on this)

## Next step

Run step 1 (`DELETE /{waba_id}/subscribed_apps`) and start the 24h clock on AC-1.
