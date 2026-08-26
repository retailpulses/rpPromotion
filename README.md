# rpPromotion

Reusable knowledge base for **Rakuten (楽天) promotion campaigns**, designed so that AI agents can quickly pick up campaign dates, registration status, step-by-step procedures, and reference materials to run promotions effectively and repeatedly.

## Purpose

- Centralize official Rakuten RMS promotion info (event dates, deadlines, registration status) per campaign.
- Store step-by-step execution guides that agents can follow to register / fix / apply for promotions.
- Accumulate reference materials (tutorial videos, official manuals, links) for both humans and agents.

## Repository Structure

| Path | Content |
|---|---|
| `guides/` | Per-campaign step-by-step guides (dates, deadlines, registration flows, action items) |
| `references/` | Reference materials: tutorials, official manuals, external links |

## How Agents Should Use This Repo

1. **Start with `guides/`** - pick the newest guide for the campaign you are working on (e.g. `2026-09-rakuten-super-sale.md`).
2. Follow the step-by-step registration / fix flows in the guide.
3. Use `references/` for tutorial videos and official manual links when a step needs more context.
4. After each campaign, **add a new guide file** under `guides/` following the same template so knowledge accumulates.

## Naming Convention

Guide files: `YYYY-MM-<campaign-slug>.md` (e.g. `2026-09-rakuten-super-sale.md`)

## Source

Info is gathered from official Rakuten RMS emails (no-reply@info.rms.rakuten.co.jp) via the store mailbox.
