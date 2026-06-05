# Session Log — 2026-06-05

## Files touched
- `SP_portal_ads_management.html` (SP Portal)
- `NSS_place_locker_ads_shipment_portal.html` (NSS Seller)
- `Shopee_place_locker_ads_seller_centre.html` (Shopee Seller)

---

## BRD — Section 2.1 (DOOH industry practices)

- Reviewed flow diagrams in images 1–3; corrected Step 1 flow chart: split location type / region diamonds → single combined diamond ("Is there ≥1 locker where location type AND region both match the ad?")
- Flow chart logic error explained: two sequential diamonds allowed false positives where different lockers satisfied each criterion separately

## BRD — Section 3.3.1 (Assigning an ad to a playlist and a locker)

Drafted full section with combined flow chart across two phases:

**Setup:** 2 criteria — location type + region. Each locker has exactly 1 of each.

**Phase 1 — Playlist assignment (at ad approval):**
- System counts lockers in each playlist where both criteria match the ad
- Playlists with ≥1 match: shown to ops pre-checked, with "X of Y lockers will show this ad"
- Playlists with 0 matches: hidden from ops

**Phase 2 — Runtime playback:**
- Each locker plays an ad only if its location type AND region both match the ad's targeting
- Partial match (one criterion only) → locker skips the ad (no over-delivery)

**Edge cases table:** partial matches, subset delivery, zero-match edge case

---

## SP Portal — `SP_portal_ads_management.html`

### Playlists tab (formerly "Schedule")
- Tab renamed: "Schedule" → "Playlists"
- Info bar: removed "Manage playlists" and "Locker coverage" buttons (duplicated in sidebar)
- Edit playback order: converted from sidebar panel → inline mode. Same button position toggles "Edit playback order" ↔ "Save playback order" (red); rows show grip icon + become draggable in edit mode; View button hidden while editing
- "Campaign PLAY ORDER" column header → plain "Campaign" (order circles already communicate this)
- View button: `btn-sm-b` → `btn-link-b` (plain blue text, no outline)
- Change playlist button: same plain blue text style; applied across all 3 locations in tab
- Spot label: `~9,000 spots` → `(estimated)`
- Legend: "spots-based (estimated end)" → "estimated end date"
- Gantt sub-line: company name shown instead of individual uploader name

### Campaign panel (`openCampPanel` — gantt View)
- Removed coloured status + type pills from top
- Creative moved to top of panel
- Field order aligned with ad review panel: Company → Uploaded by → Review date → Campaign period → Lockers (with Details expand) → Playlists → Playback order → Status → Ad type → Ad Tracking History
- Lockers cell: count + inline expand showing locker name/ID/location type/region
- `toggleCampLockers` updated to search both CAMPS + CANCELLED

### Ad review panel (`openAdPanel`)
- "Booked scope" replaced with Lockers (count + Details expand)
- Playlists and Playback order rows added
- Ad type label: "Internal" → "Internal (SPX)"
- Locker state reset on open

### Cancel campaign
- "Cancel campaign" button now opens inline reason form (textarea, min 15 chars)
- Button disabled with `opacity:0.4` + `cursor:not-allowed` until 15 chars — same UX as "Request changes"
- Cancellation reason logged as final entry in Ad Tracking History with operator + message block
- Cancelled campaigns remain visible in Ad Management tab (previously disappeared)
- `allAds` source updated to `[...CAMPS, ...CANCELLED]`
- `adStatus` and `st_label` logic fixed: Cancelled check moved before `assignedPlaylists` check to prevent false "Approved" status
- Cancelled campaigns show View (not Review) in action column
- Cancelled campaign panel: no creative review, no playlist assignment, footer = Close only

### Global
- "Review History" → "Ad Tracking History" everywhere (single change in shared `renderReviewLog`)

---

## NSS Seller — `NSS_place_locker_ads_shipment_portal.html`

### Table
- Status column: coloured pills → plain text
- Ad Info column: removed `.ai-sub` (exposure/location/period) and `.ai-tags` (status pill) — redundant with other columns
- Campaign ID column added (after Ad Info); all refs updated to `SPXSGAD` + 2-digit year + 12-digit format
- Campaign period: standardised to `YYYY-MM-DD to YYYY-MM-DD`
- Campaign period sub-text (Running, Starts in X days, Completed) removed
- Spots played column sub-text ("↑ active", "completed") removed
- Cost per 1,000 spots sub-text ("per 1,000 impr.") removed
- Cost per 1,000 spots: values updated to 2 decimal places
- "booked" sub-label under Spend → "committed"
- View button: plain blue text link; Review button: solid red

### Performance section
- "Impressions" → "Spots played" (column header, metric card, chart legend, tooltips)
- "Avg. Daily Impressions" → removed entirely; metric grid 4-col → 3-col
- "CPM" → "Cost per 1,000 spots" (column, card, tooltip updated to explain ÷ spots × 1,000)
- Green chart line (Avg. Daily) removed from SVG and legend
- SGD 83.3 → SGD 83.30

### Sidebar (View mode)
- Creative at top
- Fields: Company → Uploaded by → Review date → Campaign period → Location type → Region → Status → Ad Tracking History
- Removed: Lockers, Playlists, Playback order, Ad type (ops-internal, not useful to sellers)

### Sidebar (Review / Rejected mode)
- Location type and Region added as separate rows (replacing combined "Location" field)
- Ad Tracking History added at bottom (was missing; Shopee already had it)

### Campaign data
- All entries enriched with: `company`, `uploadedBy`, `reviewDate`, `lockers`, `playlists`, `playbackOrder`, `adType`, `locType`, `region`, `reviewLog`
- Refs updated to SPXSGAD format

### Review Now behaviour
- Fixed: `reviewNow()` now calls `goLockerAds()` first (page activation), then applies filter, then scrolls with 50ms timeout — previously only scrolled without filtering

### Cancelled campaign banner
- Added grey dismissible banner (`#cancel-banner`) separate from rejected red banner
- Rejected banner: red, not dismissible (requires action)
- Cancelled banner: grey, dismissible with × button; session-scoped via `window._cancelBannerDismissed`
- "View details" on cancelled banner filters table to Cancelled + scrolls
- Both banners can show simultaneously; rejected stacks above
- `initBanner()` updated to handle both states
- Sample cancelled campaign row + data entry added (`Q1 Trade Show 2026`)
- "Cancelled" added to status filter dropdown

---

## Shopee Seller — `Shopee_place_locker_ads_seller_centre.html`

Same changes as NSS, plus:
- "Report" button (completed row) → "View"
- `renderReviewLog` updated: heading → "Ad Tracking History" (uppercase, grey, 11px); dots → hollow grey (matching SP Portal style); timestamp layout → right-aligned 80px column with time bold on top, date below
- Sample cancelled campaign added (`11.11 Sale 2025`)

---

## Design decisions made today

| Decision | Detail |
|---|---|
| "Impressions" → "Spots played" | SPX can only track plays, not viewers; impressions implies footfall data they don't have |
| CPM → "Cost per 1,000 spots" | Consistent with spots-based model; formula = total spend ÷ spots × 1,000 |
| "Booked" → "Committed" | "Booked" doesn't communicate pending spend; "committed" is standard finance/procurement term |
| Avg. daily spots removed | Derived metric sellers can't act on; removed from both metric card and chart |
| Cancelled vs Rejected banners | Two separate banners: red (rejected, actionable, not dismissible) + grey (cancelled, informational, dismissible once) |
| Seller view sidebar fields | Lockers, Playlists, Playback order, Ad type removed — ops-internal concepts not relevant to sellers |
