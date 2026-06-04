# SPX Locker Ads — Session Summary
_29 May 2026 | Gareth Wang (gareth.wangjc@shopee.com)_

---

## Context

Self-serve digital advertising platform for SPX locker screens. Sellers book ad space; SPX ops manages campaigns, reviews creatives, and configures settings. MVP scoped to Singapore. Full BRD in `SPX_Locker_Ads_BRD_Handoff_280526.md`.

---

## 4 Portals — Current File Versions

| Portal | File | Notes |
|---|---|---|
| SPX Ops (SP Portal) | `SPX_ops_locker_ads_management_24.html` | Latest |
| NSS Seller (SPX Shipment Portal) | `NSS_seller_place_SPX_Locker_ads_v5.html` | Latest |
| Shopee Seller (Seller Centre) | `Shopee_seller_place_SPX_Locker_ads_v5.html` | Latest |
| Finance Portal | `locker_ads_rate_card.html` | No changes today |

---

## What Was Built Today

### Finance Portal — Rate Card
- New "Locker Advertisement" tab in top tab row alongside SP Commission and SP Black List
- Rate card scoped at **merchant type level** (1 rate card = 1 merchant type)
- Seller Type selected first: NSS shows merchant type dropdown; Shopee skips it
- Only configurable field: **price per spot (SGD)**
- Merchant type options follow ID-Name format: `1 - Personal`, `2 - Small Enterprise`, `3 - Big Enterprise`, `6 - Merchant Type Test 2`, `7 - Samsung Dedicated Fleet`, `8 - Samsung Normal Fleet`
- 3-step creation flow: Rate Group → Pricing Rule → Effective Time
- Table columns: Rate ID, Seller Type, Merchant Type, Price per Spot, Effective Start Time, Create Time, Update Time, Status, Action
- Shopee rate card shows `—` under Merchant Type

### Ops Portal — Booking Configuration Tab
- Renamed from "Finance" tab
- Payments and Accounts sub-tabs removed (handled in existing SPX Ops Portal)
- Rate Card & Settings collapsed into flat single view
- **Finance Portal callout banner** at top — directs ops to Finance Portal for rate management
- **Ad slot duration** — seconds per ad (editable)
- **Inactivity timeout** — seconds of locker idle before ad shows (editable); was in earlier mockup, restored today
- **Spot packages** — Package name + Total spots only (no pricing shown; price calculated at checkout from Finance Portal rate)
- Duration discounts removed (no discount config for MVP)
- Price per spot field removed from ops portal
- Formula text simplified: "Used to estimate total spots for duration-based campaigns." (removed the Σ formula)
- Location types, Regions, AI review criteria sections unchanged

### Both Seller Portals (NSS + Shopee)
- **Pricing logic fixed**: cost now `spots × PRICE_PER_SPOT` (was `lockers × weekly rate`)
- Duration cost: `weeks × EST_SPOTS_PER_WEEK × PRICE_PER_SPOT` with flat estimated spot count shown
- No formula breakdown shown to seller — just "Indicative estimate. Final cost confirmed on SPX review."
- `PRICE_PER_SPOT = 0.15` SGD (demo constant; actual rate from Finance Portal per merchant type)
- **Save as Draft removed** — flow is submit or cancel only
- **Action required callout banner** — appears below page title when campaigns need attention; shows count; "Review now" CTA scrolls to campaign list and auto-filters to Action required
- **Status filter dropdown** added (All / Pending review / Action required / Scheduled / Live / Completed / Suspended)
- Pill button filters (All / Scheduled / Live etc) removed from both files
- Draft option removed from status dropdown
- **Review history log** added to campaign detail panel — shows full timeline with dot indicators; seller sees "SPX" for ops actions; timestamps and feedback notes included

#### NSS-specific
- My Account section removed (finance and account info available in rest of SPX Shipment Portal)

#### Shopee-specific
- My Account / credits section kept (Shopee handles this natively)
- "Publish" CTA changed to "Submit for Review" (aligns with NSS)

### Ops Portal — Review Log
- `renderReviewLog()` function added — renders timeline with coloured dots (blue = seller, red = ops, grey = system)
- Injected into `openAdPanel(ref)` — the real function called from Ad Management tab
- All 6 CAMPS entries have `reviewLog` data
- Ops view shows full attribution (`ops · gareth.wangjc`); seller view masks to "SPX"
- `p-body` padding-bottom increased so log clears sticky footer

---

## Key Decisions Made Today

| Decision | Rationale |
|---|---|
| Rate card at merchant type level, not per company | Phase 1 simplicity; follows legacy structure |
| Shopee treated as one flat bucket | SPX has no visibility into individual Shopee seller accounts |
| No draft state | Short form; no meaningful reason to hold drafts for MVP |
| No Save as Draft | Either submit or cancel |
| Duration billing trigger = campaign completion date | Actual spots only known at end |
| Spots billing trigger = campaign creation date | Price locked and known at booking |
| NSS My Account removed from ads module | Already available elsewhere in shipment portal |
| Pill status filters replaced by dropdown | 6+ statuses makes pills unwieldy |
| Review log always visible (even on first submission) | Industry standard (Meta, Google Ads); sets expectation from day one |
| Ops identity masked to "SPX" on seller-facing log | Seller doesn't need staff names |

---

## Canonical Status List (from flowchart)

`Draft` → `Pending review` → `Action required` → `Scheduled` → `Live` → `Completed` → `Suspended` / `Cancelled`

All four portals now use title case consistently. Ops portal previously used lowercase (`live`, `scheduled`, `complete`, `cancelled`) — fixed to match.

---

## Open Items / Pending

| Item | Notes |
|---|---|
| Finance Portal — duplicate rate card guard | BRD to specify backend logic (prevent two active rate cards for same merchant type); reference existing logistics rate card logic |
| Flaw 4 (ops review context) | Ops can't distinguish first submission from resubmission at status level. Mitigated by review log showing history. Worth noting in BRD for eng. |
| Shopee seller billing spec | Shopee reports charges; SPX doesn't invoice seller directly. BRD note: billing statement adds locker ads line item |
| NSS billing entry point | NSS charges bundled into existing monthly SBC/EBS cycle. BRD note: link to existing invoice view |
| QR code tracking for creatives | Not in MVP scope |
| Operating hours field in SP Management | Needed for accurate spot estimation in production |
| Finance tab UI (ops portal) | Read-only charges summary — pending billing template field spec from Piao Quan / finance eng |
| NSS flow payment alignment | Pending: Rosemary Tang, Rui Xuan, Goh Yi Lin |
| Cross-market config | Post-SG MVP |
| Tooltip / FAQ links | "What is SPX Locker Ads?" and "Learn more" on create page — local teams to provide links |
| Seller mockups tomorrow | Gareth to share NSS and Shopee seller mockups for further review |

---

## Architecture Summary

```
Finance Portal          → Sets price per spot per merchant type
        ↓
SPX Ops Portal          → Configures spot packages, location types, ad slot duration, inactivity timeout
        ↓
Seller (NSS / Shopee)   → Books campaign (spots or duration), uploads creative
        ↓
Ops reviews creative    → AI screening + human approval; review log visible to both sides
        ↓
Ops assigns playlist    → System suggests based on booked location types + regions
        ↓
Campaign goes Live      → Locker screens play ad
        ↓
Billing                 → Spots: invoiced at creation date | Duration: invoiced at completion
```
