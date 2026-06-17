# Session Log — 2026-06-16

## Files touched
- `NSS_place_locker_ads_shipment_portal.html` (NSS Seller)
- `Shopee_place_locker_ads_seller_centre.html` (Shopee Seller)
- `SP_portal_ads_management.html` (SP Portal)
- Memory: `project_locker_ads.md` updated with finalized billing decisions

---

## BRD — Section 3.6 Billing Logic (finalized)

### Core billing formula
Both exposure types use the same unit rate: **Spots × Price per Spot (SGD)**. Price per spot is set in Finance Portal rate card by merchant type.

### By Spots
- Invoice = **Booked Spots × Price/Spot** (fixed, known at order placement)
- Pay-by-order → invoice on **campaign go-live date**
- Pay-by-cycle → invoice in **end-of-month cycle when campaign ends**
- Amount is fixed; no post-campaign reconciliation needed

### By Duration
- Invoice = **Actual Spots Delivered × Price/Spot** (known only at campaign end)
- No cap on over-delivery — seller pays for all spots received (aligned with market practice)
- Pay-by-order → **immediate invoice after campaign ends**
- Pay-by-cycle → invoice in **end-of-month cycle when campaign ends**
- "Estimated minimum" shown at booking for transparency only — **not a commitment, no refund/make-good** (Option A)

### Invoice timing decision — go-live, not approval
Billing triggers at **campaign go-live**, not at order placement or ops approval. Reasoning: digital self-serve platforms (Google Ads, Meta, programmatic DOOH) charge when service activates, not before. Billing at approval would charge sellers before a single ad plays, which breaks the "bill when services are completed" principle — especially for campaigns booked weeks in advance.

### Pay-by-order vs pay-by-cycle framework
Existing seller attribute inherited by ads billing. Key clarification: pay-by-order's "bill immediately at order" rule does **not** apply to By Duration campaigns, since the final amount is unknown until campaign end. Duration campaigns always bill at service completion regardless of seller type.

---

## BRD — Estimated Spots Formula (By Duration)

```
Est. Spots = Campaign Days × Est. Daily Plays per Locker × Eligible Locker Count
```

| Variable | Definition |
|---|---|
| Campaign Days | End date − Start date (min. 7 days) |
| Est. Daily Plays per Locker | Ops-configured constant in SP Portal Ad Settings |
| Eligible Locker Count | Commissioned lockers matching seller's Location Type AND Region (nightly snapshot) |

**Decisions:**
- Active Hours / Slot Duration / Playlist Size formula dropped — too many variables across markets (SG, MY), locker types, and opening hours; also doesn't account for transactional downtime
- Est. Daily Plays per Locker is a single constant set by ops via SP Portal — updated periodically as delivery data matures
- Eligible locker count uses nightly snapshot; "Island-wide" region = all commissioned lockers for the selected location type(s)
- Minimum campaign duration: **7 days** (below this, estimate is too small to be meaningful)
- Post-MVP: replace ops constant with 30-day rolling average of actual delivery data

---

## BRD — Post-rejection edit policy

Post-rejection, sellers can **only re-upload a revised creative** and resubmit. Campaign info (dates, location type, region) cannot be changed through the rejection flow.

**Rationale:** Aligned with industry practice (Google Ads, Meta, DOOH platforms). Rejection is a creative-level event — ops reviewed the creative, not the campaign parameters. Mixing campaign edits into the rejection flow creates ambiguity about what ops reviewed and signed off on.

**If campaign details need changing:** seller must cancel the campaign and submit a new one.

**Edge case to document in BRD:** If SPX rejects for a reason requiring campaign info changes (e.g., unsupported region), ops must explicitly state in rejection remarks: *"Please cancel this campaign and submit a new one with updated details."*

---

## Code changes — NSS Seller Portal

### 1. By Duration radio card description
- Removed: *"A minimum number of spots is guaranteed — any additional plays are a bonus."*
- Replaced with: *"Your ad runs for the full campaign period. An estimated number of spots is shown at booking — final charge is based on actual spots delivered."*

### 2. Edit permissions
- Fixed missing `onclick` on Pending Review table row (existing bug — panel couldn't open)
- Edit campaign button now shown for `Pending review` status (was incorrectly showing for `Scheduled`)
- Post-approval statuses (Scheduled, Live, Completed, Cancelled, Suspended) are view-only
- Edit panel warning copy updated: "Saving changes will reset the review — make sure ops can review in time"

### 3. Estimated spots formula wired to UI
- Location Type and Region chips tagged with `data-group="region"` / `data-group="loctype"`
- Replaced `EST_SPOTS_PER_WEEK` with `EST_DAILY_PLAYS = 200` + `LOCKER_COUNTS` lookup table (Region × Location Type)
- `calcDurCost()` now computes: `days × EST_DAILY_PLAYS × getEligibleLockers()`
- Shows `—` with prompt if region or location type not yet selected
- `tog()` triggers `calcDurCost()` on every chip toggle when By Duration is active

### 4. Edit panel — Location type + Region as chips
- Replaced free-text inputs with chip selectors matching the create form options
- Chips pre-select the campaign's current values on open ("All types" pre-selects all location type chips)
- `saveNSSEdit()` reads from `[data-group="edit-loctype"].on` and `[data-group="edit-region"].on`

### 5. Edit panel — Creative section improvements
- Upload zone: replaced plain `uzone` with styled `upload-zone` class (☁️ icon, hover effect)
- Added "What SPX reviews" guidelines box below upload zone (same as create form)
- Cancel + Save changes buttons moved to **sticky footer** (`panel-footer` div, fixed at bottom of sidebar, outside scrollable body)

---

## Code changes — Shopee Seller Portal
All changes mirror NSS (same logic, Shopee colour variables):
- By Duration radio card description updated
- Edit button logic: Pending review = editable, all others = view-only
- Estimation formula wired to chips
- Edit panel chips for Location type + Region
- Edit panel creative section improved + sticky footer

---

## Code changes — SP Portal

### Ad Settings — new config field
- Added `est_daily_plays: 200` to `RC` config object
- Added "Est. daily plays per locker" number input in Ad Settings, after Inactivity timeout
- Hint text explains purpose and that ops should update periodically as delivery data becomes available
