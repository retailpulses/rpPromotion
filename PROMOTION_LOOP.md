# Promotion Loop Definition

**Status:** Draft for review  
**Version:** 0.1.0  
**Scope:** Amazon, Rakuten, Mercari Shops  
**Timezone:** Asia/Tokyo

## Purpose

Maintain a current, evidence-backed promotion radar and turn material marketplace announcements into prepared, executed, and reviewed campaigns.

This document defines the operating loop. It does not enable an automation or authorize marketplace, customer, pricing, advertising, inventory, or database writes.

## Loop

```text
Discover → Normalize → Qualify → Update radar → Plan → Execute → Monitor → Review
    ↑                                                                    │
    └────────────────────────────── Learn ────────────────────────────────┘
```

### 1. Discover

Collect new promotion information since the last successful scan.

Primary sources:

- official Amazon seller emails and Seller Central announcements;
- official Rakuten RMS emails and 店舗運営Navi;
- official Mercari Shops merchant emails and promotion/news pages.

Default cadence:

- weekly while no L1 campaign is active or approaching;
- daily when an L1 campaign is inside its active preparation or execution window;
- event-driven when an official correction, postponement, cancellation, or deadline notice arrives.

The scanner must retain a watermark using the source plus Message ID or stable announcement identifier. Re-running the loop must not create duplicate evidence or actions.

### 2. Normalize

For each new source, record:

- marketplace;
- source type and stable source identifier;
- received/published time in JST;
- campaign name and edition;
- explicitly stated campaign dates;
- submission, inventory, creative, pricing, or advertising deadlines;
- eligibility, fees, discount rules, and material changes;
- source file or URL;
- extraction limitations.

Preserve readable source evidence. Redact passwords, tokens, credentials, customer information, and unrelated order/support content before committing anything to Git.

### 3. Qualify

Classify the announcement:

| Level | Meaning | Default response |
|---|---|---|
| L1 | Strategic marketplace-wide event with material traffic/revenue impact | Separate dated workspace and full campaign lifecycle |
| L2 | Meaningful but bounded campaign or opportunity | Assess economics and open a workspace when justified |
| L3 | Recurring/tactical promotion | Update persistent guidance; avoid unnecessary per-edition workspaces |
| Ignore | Operational, educational, unrelated advertising, duplicate, or non-actionable | Retain only when needed as source evidence |

Promotion status:

- `Expected`: historical or recurring pattern; exact future date is not officially confirmed.
- `Confirmed`: an official source explicitly states the event or deadline.
- `Changed`: a later official source modifies an earlier confirmed fact.
- `Cancelled`: an official source cancels the event.
- `Closed`: the event and required review are complete.

Only official marketplace evidence may move a date from `Expected` to `Confirmed`, `Changed`, or `Cancelled`.

### 4. Update radar

Update the campaign radar with:

- evidence status and source pointer;
- event dates and all separately stated deadlines;
- L1/L2/L3 level;
- preparation window;
- next action, owner, and due date;
- conflict or correction history.

Never silently overwrite an earlier confirmed date. Record the superseding source and the change.

### 5. Plan

When a campaign enters its preparation window:

- create or update the campaign workspace;
- confirm assortment, margin, stock, inbound supply, and replenishment deadlines;
- identify platform submission and setup requirements;
- assign owners and escalation conditions;
- record assumptions separately from verified current facts.

Opening a workspace is automatic documentation work. Commercial decisions remain subject to the approval gates below.

### 6. Execute

Execution follows the approved campaign plan. Before any external write, verify the current platform rule, target account/shop, dates, prices, inventory, budget, and rollback or correction path.

### 7. Monitor

During the campaign, prioritize exceptions:

- incorrect or missing promotion status;
- unexpected price/discount behavior;
- stockout or replenishment risk;
- material traffic/conversion deviation;
- budget or margin breach;
- fulfillment or customer-impacting problems;
- new official corrections or deadline changes.

### 8. Review and learn

After the campaign:

- verify promotion shutdown;
- record traffic, conversion, orders, revenue, contribution, promotion cost, stock, and fulfillment outcomes where available;
- document failures, missed deadlines, and reusable learning;
- close the campaign only after required review is complete;
- promote only stable, reusable learning into shared guidance.

The next discovery cycle uses these learnings but must still re-verify time-sensitive platform rules.

## Approval boundaries

The loop may perform read-only discovery, deduplication, evidence archiving, radar updates, and draft planning without separate approval.

Explicit approval is required before:

- changing live marketplace prices, discounts, coupons, points, ads, listings, or inventory;
- submitting or registering a promotion;
- sending marketplace, supplier, or customer messages;
- committing advertising or coupon budget;
- changing production databases or operational systems;
- cancelling or materially changing an approved campaign.

Urgency does not bypass these gates.

## Outputs per successful cycle

- scan timestamp and source coverage;
- previous and new watermarks;
- new, changed, cancelled, duplicate, ignored, and failed-source counts;
- archived evidence for accepted announcements;
- radar changes with source pointers;
- newly created or updated campaign workspaces;
- action list with owner, due date, level, and approval requirement;
- unresolved extraction or source-access failures.

If no material change is found, record a compact no-change result rather than rewriting the annual archive.

## Cycle acceptance criteria

A cycle is complete only when:

- every configured source either succeeded or has a visible failure;
- all discovered records have stable identifiers;
- duplicate Message IDs/source identifiers are zero in newly written evidence;
- all confirmed dates and deadlines have source pointers;
- secrets and unrelated personal/order information are absent from committed files;
- changes and cancellations supersede earlier facts without erasing history;
- actions identify whether approval is required;
- the watermark advances only after successful persistence.

## Proposed initial operating policy

- Run the read-only discovery cycle every Monday at 09:00 JST.
- Escalate to a daily 09:00 JST scan for an active or approaching L1 campaign.
- Review the resulting radar/actions before enabling any external-write automation.
- Keep the annual email archive as a baseline; append only new evidence and update summaries incrementally.

## Review questions

1. Is Monday 09:00 JST the correct weekly cadence?
2. Should daily L1 scanning begin at the radar window or the active preparation window?
3. Who owns triage and deadline acknowledgement for each marketplace?
4. Should radar updates be committed automatically, or proposed through a pull request?
5. Where should the durable watermark and cycle-run history live?
6. Which metrics are mandatory before an L1/L2 campaign can be closed?
