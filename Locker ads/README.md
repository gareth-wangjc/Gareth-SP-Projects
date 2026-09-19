# SPX Locker Ads — Project Overview

_Last updated 2026-09-19. This file is meant to be kept current — update it whenever a session changes the file map, the status flow, or a locked decision, so picking this project back up doesn't require re-reading every session log._

## What this is

Self-serve digital OOH ad platform for SPX locker screens. Sellers (NSS + Shopee) book ad space on locker-screen playlists; SPX ops reviews creatives, assigns playlists, and manages campaigns. MVP scoped to Singapore. Everything in this folder is BRD-support material: HTML mockups (no backend, no build step, static seed data) used to pin down UX/logic before writing the PRD, plus session logs recording *why* each decision was made.

**Why this project exists:** SG upper management started requesting manual ad uploads via backend since Apr 2026; needed a proper ops interface. Also strategic parity with 3PL locker ad providers (e.g. Pick! Locker).

**Full BRD:** Google Doc `1cdgkkzKLqXkLUDTacb9uAdL2ksUSxBKtwZv78nnJgII`. **Phase 2 PRD:** Confluence pageId `3295849790` (Gareth writes this by hand; mockups + session logs are the working notes behind it).

## The 4 portals — current files

| Portal | File | Who uses it |
|---|---|---|
| SPX Ops (SP Portal) | `SP_portal_ads_management.html` | SPX ops — review/approve campaigns, configure booking rules, assign playlists |
| NSS Seller | `NSS_place_locker_ads_shipment_portal.html` | Non-Shopee sellers, via SPX Shipment Portal |
| Shopee Seller | `Shopee_place_locker_ads_seller_centre.html` | Shopee sellers, via Seller Centre |
| Finance Portal | `finance_portal_ads_rate.html` | Finance — rate cards, GST, billing config |

These are **separate static files, not a shared live app** — config that's conceptually shared (region lists, coverage packages, rate cards) is duplicated as a hardcoded snapshot in each file, always with a code comment noting it'd be a live API pull in production. Don't assume editing one updates another.

Also in this folder: `locker-advertisements-phase-1-DRAFT.html` (early draft, superseded), `SP Commission Fee Tiered Mass Template.xlsx` (real reference template the Finance mass-create GSheet is modeled on), `User guide images/` (screenshots for the ops user guide).

## Campaign lifecycle (current, as of 2026-09-19)

```
Seller submits (uploads creative + booking)
        ↓
Pending review  ──[any reviewer rejects, any time]──→ Rejected → back to seller
        │  (SPX ops: AI screening advisory + human "approval groups" sign-off — see below)
        ↓  (all configured approval groups have signed off)
Approved  ──(zero eligible playlists)──→ stays Approved, ops finishes via side-panel "Assign playlist"
        ↓  (eligible playlists auto-assigned)
Pending payment  ──[unpaid past window]──→ auto-cancelled
        ↓  (seller pays via NSS QR/Sender Balance)
Scheduled / Live  (during campaign period)
        ↓
Completed
```

`Cancelled` is a separate terminal state reachable from most non-terminal statuses via ops's "Cancel Campaign" action (reason required).

**Approval groups (added 2026-09-19):** Booking Configuration → "Ad review approvals" lets ops configure 1–3 named groups (default: Ops / Biz Lead / Manager), each with a roster of approvers. A campaign only reaches `Approved` once every currently-configured group has ≥1 sign-off — any member of a group can sign for it, any order, no enforced sequence. This was added because SG local ops wanted 2–3 layers of sign-off; it's expected to be temporary (DOOH norm is 1 layer), so reverting is deleting groups down to 1 in Booking Configuration — no code change needed. Full detail: `session_log_2026-09-19.md`.

## Key decisions currently in force

| Area | Current rule |
|---|---|
| Billing rate | Per locker **per day** (not week/spot). Booking is in whole weeks (`weeks × 7` days billed) — booking granularity ≠ billing granularity, chosen so pro-rata refunds are trivial. |
| Coverage packages | Ops-configurable (`ADS_COVERAGE`/`AD_COVERAGE`), e.g. Half/Full — % of the seller's own *matched* lockers, not of the network. Configured in SP Portal Booking Configuration. |
| Rate card | Finance Portal — one base rate grid (Location Type × Region → SGD/locker/day) **plus** a separate Fee Table discount ladder keyed on the *coverage %* (not volume-of-network %). Versioned as full snapshots; a campaign locks to the version active on its booking date. |
| Cost formula | Per (Region × Location Type) combo: `billed = ceil(matched × coverage%)` (round **up**), `subtotal = billed × rate × 7 × weeks`; sum combos, apply one discount % (from the coverage-% band), apply GST, round once at the end (half-up, 2dp). |
| Playlist assignment | Auto-matched by locker-level intersection (Location Type **and** Region both match) the moment a campaign is approved — always **before** payment, never gated by it. A per-locker cap (`RC.max_seller_ads_per_locker`) excludes fully-saturated playlists and flags partially-saturated ones. |
| Payment | Seller pays via NSS (Sender Balance / QR) within a configurable window (`RC.payment_window_hours`, default 24h) after playlist assignment. No ops-side override to mark paid. Unpaid on timeout → auto-cancel. |
| AI screening | Advisory only — a flag never blocks or gates approval. Human sign-off is always the final gate. |
| Approval | Multi-group sign-off threshold, see lifecycle above. |
| Refunds | Handled offline (credit note), not in-system — this is what keeps "1 campaign = 1 transaction" true for Finance. |

Superseded/retired, in case old context surfaces: by-spots billing, Individual Locker package, external enterprise clients (Singtel/Gov — deferred), the old compass-region taxonomy (replaced by CCR/RCR/OCR), "Network"-named coverage tiers (renamed "Coverage").

## Where to look for more

- **`session_log_YYYY-MM-DD.md`** — one per work session, in chronological order, each self-contained with a "Context" header. Read the most recent 2–3 to see what's actively in flux.
- This README — current-state snapshot only. If it and a session log disagree, the **newer session log wins**; update this file to match.
- Confluence PRD (pageId `3295849790`) and BRD Google Doc — the actual deliverables; these mockups exist to de-risk decisions before they're written there.

## Known open items (as of 2026-09-19)

- NSS's `AD_RATES` (half/full-keyed) not yet reconciled to the Location Type × Region rate-band shape used elsewhere.
- §3.4.3 PRD wording still has an exactly-5-available boundary gap in its worked example (general rule is fixed).
- §3.6.5 pro-rata refund formula not yet drafted in the PRD.
- Two open PRD questions on how refund transactions should display (§3.6.2/§3.6.3).
- Mass Create GSheet template needs the Flat Fee/Tiered label-swap wording applied (given to Gareth as copy-paste text, not yet confirmed applied).
- Seed data: "FairPrice Q2" is mislabeled as pending-review in an old comment but is actually past that stage — doesn't exercise the new approval-groups flow.
- Real notification mechanism (SeaTalk bot on submission) and real identity/SSO binding for approval sign-off are both PRD-level dev asks, deliberately out of scope for these static mockups.

## Working conventions for this project

- Verification bar: headless Playwright click-through after any logic change, not just visual inspection (cached copy under `%LOCALAPPDATA%\npm-cache\_npx\e41f203b7505f1fb\node_modules` via `NODE_PATH` if no global install).
- Model routing (Gareth's instruction): open-ended design discussion → Opus 5 high; implementation → Sonnet 5 high.
- Gareth writes the actual PRD/BRD by hand — deliverables from sessions here are terse bullets/tables/formulas/worked examples he pastes in himself, not prose he'd need to rewrite.
- When a config value is duplicated across the 3-4 portal files (rate cards, region lists, coverage packages), that's a deliberate mockup limitation, always flagged in a code comment — don't silently let one file drift from another without noting it.
