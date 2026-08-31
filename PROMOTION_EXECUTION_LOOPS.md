# Promotion Execution Loops

**Process-map levels:** L2 and L3  
**Status:** Draft for review  
**Version:** 0.1.0  
**Scope:** Amazon, Rakuten, Mercari Shops  
**Timezone:** Asia/Tokyo

## Purpose

Define small, stateful loops that agents can run end-to-end. Each loop has a trigger, machine-readable inputs, deterministic decisions, bounded actions, verification, recovery, and an auditable terminal state.

These definitions do not schedule a job or authorize a live marketplace action. They define how automation should behave after the operating policy is approved.

## Automation modes

Human-in-the-loop is a policy choice, not a mandatory step in every run.

| Mode | Agent behavior | Human role |
|---|---|---|
| `observe` | Read, classify, archive, report; no state-changing action | Reviews output when useful |
| `supervised` | Complete analysis and dry-run; pause only at a gated action | Approves/rejects exact proposed action |
| `bounded-autonomous` | Execute actions inside a pre-approved policy envelope and verify readback | Reviews exceptions and policy changes |

No mode may expose secrets or silently broaden scope. A policy envelope must specify shop/account, action types, value/budget bounds, date window, eligible campaign level, rollback/correction path, and notification rules.

## Loop portfolio

| ID | L2 execution loop | Trigger | Autonomous terminal outcome |
|---|---|---|---|
| P1 | Promotion email → promotion action | New official email or scheduled mailbox scan | Evidence archived; radar/action/workspace updated; approved action verified or queued |
| P2 | Radar entry → campaign workspace | Campaign enters preparation window | Correct workspace created/updated with owners and readiness checklist |
| P3 | Deadline → acknowledgement/escalation | Deadline created or threshold crossed | Owner/action state updated; overdue risk escalated |
| P4 | Readiness → exception resolution | Scheduled readiness check or source change | Passing checks recorded; exceptions assigned or bounded fix verified |
| P5 | Live promotion → operational exception | Campaign active or live source event | Live state verified; bounded correction applied or escalated |
| P6 | Campaign end → close/review/learning | End time crossed and shutdown verified | Review completed, learnings routed, campaign closed |

P1 is the intake loop. P2–P6 consume actions and state produced by P1.

---

## P1 — Promotion email to promotion action

### Outcome

Turn a new official platform email into one of five verified outcomes:

1. duplicate safely ignored;
2. evidence archived with no action;
3. radar fact created or updated;
4. owned internal action/workspace created or updated;
5. exact external action executed within policy, verified, and reconciled — or placed in an approval/exception queue.

### Trigger

- mailbox webhook/event, when available; or
- scheduled incremental scan using the last successful watermark.

Recommended initial schedule: every Monday at 09:00 JST, increasing to daily for an approaching/active L1 campaign.

### Input contract

```yaml
run:
  run_id: string
  mode: observe | supervised | bounded-autonomous
  started_at_jst: datetime
  previous_watermark: string | null
source:
  platform: amazon | rakuten | mercari
  account: string
  folder_scope: [string]
policy:
  version: string
  allowed_senders: [string]
  allowed_actions: [string]
  shops: [string]
  value_bounds: object
  active_window: datetime-range | null
```

Secrets are runtime references, never input values persisted in the run record.

### L3 workflow

```text
Scan incrementally
  → authenticate sender/source
  → fetch full message
  → compute idempotency key
  → detect duplicate/superseding message
  → extract campaign facts and deadlines
  → validate dates, marketplace, shop and source confidence
  → classify L1/L2/L3/Ignore and change type
  → archive redacted evidence
  → update radar/history
  → derive actions
  → route each action through policy
  → dry-run or execute
  → read back authoritative state
  → reconcile action/run state
  → advance watermark
  → notify only on material change, exception or requested digest
```

### State machine

```text
DISCOVERED
  → AUTHENTICATED
  → PARSED
  → QUALIFIED
  → EVIDENCE_SAVED
  → ACTIONS_DERIVED
      ├─→ NO_ACTION → CLOSED
      ├─→ INTERNAL_UPDATE → VERIFIED → CLOSED
      ├─→ WAITING_APPROVAL → APPROVED/REJECTED/EXPIRED
      └─→ EXECUTING → VERIFIED → CLOSED

Any state → RETRYABLE_FAILURE | QUARANTINED | MANUAL_EXCEPTION
```

The state transition and its evidence must be persisted atomically. The watermark advances only after all messages up to that watermark reach a durable terminal or exception state.

### Idempotency and supersession

Primary idempotency key:

```text
zoho_account_id + folder_id + message_id
```

Secondary content fingerprint:

```text
normalized_sender + normalized_subject + platform + normalized_campaign_edition + body_hash
```

- The Message ID prevents replay of the same email.
- The fingerprint identifies near-duplicate broadcasts to multiple aliases.
- A correction, postponement, cancellation, or changed deadline is a new event that supersedes — never deletes — the prior fact.
- An external action gets its own operation ID and must be read back before retrying an ambiguous result.

### Extraction contract

```yaml
promotion_event:
  event_id: string
  platform: amazon | rakuten | mercari
  campaign_name: string
  edition: string | null
  level: L1 | L2 | L3 | Ignore
  change_type: new | reminder | correction | postponement | cancellation | duplicate
  status: Expected | Confirmed | Changed | Cancelled | Closed
  event_start_jst: datetime | null
  event_end_jst: datetime | null
  deadlines:
    - type: submission | inventory | creative | pricing | advertising | other
      due_at_jst: datetime
      explicit_in_source: true
  affected_shops: [string]
  commercial_terms: object
  source_evidence_path: string
  source_message_id: string
  extraction_confidence: high | medium | low
  unresolved_fields: [string]
```

An inferred date must not be written as confirmed. Conflicting explicit dates force `Changed` or `MANUAL_EXCEPTION`, depending on source authority.

### Action derivation rules

| Condition | Derived action |
|---|---|
| Duplicate with no new fact | Close as `NO_ACTION` |
| Informational L3 with reusable rule | Update persistent guidance/radar |
| New explicit deadline | Create/update owned deadline action |
| L1 confirmed or changed | Create/update dated campaign workspace and readiness actions |
| L2 confirmed | Run economic/operational qualification; open workspace if threshold passes |
| Cancellation/postponement | Freeze pending execution, update radar/workspace, notify owner |
| Eligible configuration request | Produce exact dry-run; execute only if policy permits |
| Missing body, ambiguous shop/date, failed source authentication | Quarantine; do not invent an action |

### Policy routing

Actions are divided by impact, not by implementation difficulty.

**Autonomous by default:**

- read email and official source;
- archive redacted evidence;
- deduplicate and classify;
- update internal radar/history;
- create or update internal campaign documents and action drafts;
- send a configured internal digest.

**Autonomous only inside an approved envelope:**

- create internal reminders/tasks for named owners;
- configure a previously approved promotion with exact shop/SKU/date/value bounds;
- apply a reversible correction explicitly permitted by policy;
- send a pre-approved internal operational notification.

**Always gate unless separately pre-authorized by an explicit policy:**

- price, discount, coupon, points, ad-spend, listing, or inventory changes;
- promotion registration/submission;
- supplier, marketplace, or customer messages;
- database/production writes;
- cancellation or material change of an approved campaign.

In `supervised` mode, the approval packet must contain the exact target, before/after state, source evidence, dry-run result, risk/budget impact, verification method, and expiry time.

### Verification

Internal updates:

- source pointer resolves;
- radar/workspace contains the new fact and preserved history;
- action owner and due date are present;
- no duplicate action is created.

External actions:

- read back from the authoritative marketplace/API/UI;
- compare exact shop, campaign, SKU scope, dates, price/discount and status;
- record operation ID and readback evidence;
- treat timeout or ambiguous response as unknown, not success;
- do not retry until authoritative readback proves the action absent or failed.

### Failure and recovery

| Failure | Recovery |
|---|---|
| Mail/API authentication failure | Keep watermark unchanged; retry with backoff; alert after threshold |
| Full body unavailable | Preserve metadata; attempt supported content fetch; quarantine if still incomplete |
| Parser confidence low | Archive source; create no live action; queue exception |
| Unknown/ambiguous shop | Stop before action; request mapping resolution |
| Conflicting official dates | Preserve both sources; mark changed/conflict; require authority resolution |
| Internal write failure | Retry idempotently; do not advance watermark |
| External response ambiguous | Read back official state before any retry |
| Verification mismatch | Stop subsequent actions; attempt only approved rollback/correction; escalate |

### Run output contract

```yaml
result:
  run_id: string
  status: complete | complete_with_exceptions | failed
  previous_watermark: string | null
  new_watermark: string | null
  discovered: integer
  duplicates: integer
  archived: integer
  radar_changes: integer
  actions_created: integer
  actions_executed: integer
  actions_verified: integer
  waiting_approval: integer
  quarantined: integer
  exceptions: [object]
  evidence_paths: [string]
```

### Definition of done

P1 is complete when every discovered email is durably duplicate, closed, waiting approval, or quarantined; accepted evidence is redacted and linked; all derived actions are idempotent and owned; every executed action has authoritative readback; exceptions are visible; and the watermark advances only through the fully persisted range.

---

## P2 — Radar entry to campaign workspace

### Trigger

A confirmed L1 event, a qualified L2 event, or an existing campaign entering its preparation window.

### Agent loop

1. Resolve campaign identity and dated edition.
2. Detect an existing workspace by campaign key.
3. Create from the approved template or update in place.
4. Populate official dates, deadlines and source pointers.
5. Create readiness actions for assortment, margin, inventory/inbound, supplier terms, platform setup, measurement and ownership.
6. Verify required files/fields and link the radar entry.

### Terminal states

`WORKSPACE_READY`, `UPDATED`, `WAITING_DECISION`, or `QUARANTINED`.

No SKU, pricing, inventory or budget decision is inferred merely to complete the workspace.

## P3 — Deadline to acknowledgement and escalation

### Trigger

New/changed deadline, T-minus threshold, missing acknowledgement, or overdue state.

### Agent loop

1. Deduplicate by campaign + deadline type + due time + shop.
2. Calculate JST thresholds and business-time remaining.
3. Confirm owner and required evidence.
4. Create/update internal action.
5. Notify according to severity and policy.
6. Verify acknowledgement or completed evidence.
7. Escalate only when thresholds are crossed or ownership is missing.

### Terminal states

`ACKNOWLEDGED`, `COMPLETED`, `SUPERSEDED`, `OVERDUE_ESCALATED`, or `OWNER_MISSING`.

## P4 — Readiness to exception resolution

### Trigger

Scheduled readiness check, changed campaign fact, or approaching execution gate.

### Agent loop

1. Read current campaign workspace and approved operational sources.
2. Check assortment decision, stock/inbound, supplier terms, contribution, setup status, owners and measurement baseline.
3. Record pass/fail/unknown per check with evidence timestamp.
4. Create exceptions for failed/unknown critical checks.
5. Apply only bounded approved fixes.
6. Re-read state and close verified exceptions.

### Terminal states

`READY`, `READY_WITH_ACCEPTED_RISK`, `BLOCKED`, or `WAITING_APPROVAL`.

## P5 — Live promotion to operational exception

### Trigger

Campaign start/end threshold, scheduled live check, marketplace event, or metric/stock anomaly.

### Agent loop

1. Read authoritative promotion status and intended configuration.
2. Compare shop, SKU scope, time, price/discount, budget and inventory.
3. Classify normal state versus exception.
4. Apply a reversible correction only inside policy.
5. Read back and reconcile exact state.
6. Notify on material exception, failed correction or policy breach.

### Terminal states

`HEALTHY`, `CORRECTED_AND_VERIFIED`, `WAITING_APPROVAL`, or `ESCALATED`.

## P6 — Campaign end to review and learning

### Trigger

Official end time crossed and promotion shutdown verified.

### Agent loop

1. Verify no unintended discount/promotion remains live.
2. Collect approved performance and cost sources.
3. Compare plan, baseline and actual results.
4. Record exceptions and missed deadlines.
5. Draft reusable learning and route it for shared promotion only when stable.
6. Close actions and campaign after required review fields pass.

### Terminal states

`CLOSED`, `REVIEW_INCOMPLETE`, `SHUTDOWN_EXCEPTION`, or `DATA_UNAVAILABLE`.

## Cross-loop control plane

All loops should share:

- a stable campaign/event identifier;
- append-only source and state-transition history;
- operation IDs for state-changing actions;
- a policy version captured on each decision;
- an exception queue with owner, severity and next retry/action;
- a run ledger with input watermark, output watermark and verification evidence;
- replay-safe handlers and bounded retries;
- notification suppression for unchanged healthy state.

## Recommended implementation sequence

1. Implement P1 in `observe` mode: email → evidence → radar/action draft.
2. Add P2 and P3 for autonomous internal documentation and deadlines.
3. Add P4 read-only readiness checks and exception routing.
4. Run P1–P4 in shadow mode and measure precision, duplicates, missed deadlines and exception quality.
5. Enable `supervised` exact-action packets for selected marketplace actions.
6. Enable `bounded-autonomous` actions one action type/shop at a time after successful canaries and readback verification.
7. Add P5 and P6 using the same state, operation and evidence contracts.

## Alignment questions

1. Should P1 create internal tasks automatically, or only update repository action files initially?
2. What system should own the durable watermark, run ledger and exception queue?
3. Which marketplace/shop should be the first `supervised` canary?
4. Which exact actions, if any, may enter the first bounded-autonomous policy envelope?
5. What confidence threshold permits automatic radar updates versus quarantine?
6. Who owns each platform's exceptions and approval expiry?
