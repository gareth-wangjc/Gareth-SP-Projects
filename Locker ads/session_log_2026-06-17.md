# Session Log — 2026-06-17

## Files touched
- `NSS_place_locker_ads_shipment_portal.html`
- `Shopee_place_locker_ads_seller_centre.html`
- `SP_portal_ads_management.html`

---

## NSS + Shopee: CTA button right-alignment (carried over from 2026-06-16)
- Added `justify-content:flex-end` to `panel-footer` / `sh-panel-footer` static div
- Removed duplicate `display:none` from same divs
- Removed `flex:1` from "Save changes" and "Confirm cancellation" buttons in JS-generated footer innerHTML so buttons sit at natural width when right-aligned

---

## NSS + Shopee: Campaign start date validation
- **Rule**: Start date must be at least 7 days from today to allow time for review and scheduling (BE-configurable per BRD note)
- **Create form**: Added `<div id="sd-err">` below date-row; `calcDur()` now checks start date on every change and shows/hides error inline; `doPublish()` blocks submission if check fails
- **Edit panel**: Added `onchange="checkNSSEditStart()"` / `checkSHEditStart()` on start date input; error div injected below date inputs in dynamically-generated HTML; `saveNSSEdit()` / `saveSHEdit()` re-validates before saving
- **Wording**: *"Start date must be at least 7 days from today to allow time for review and scheduling."* (Google Ads / Meta Ads Manager pattern)

---

## SP Portal: Priority Actions — Pending schedule tab
- Added `pendingSchedule` array: campaigns with `approvedDate` set but `assignedPlaylists.length === 0`, not rejected/cancelled
- Added "Pending schedule" filter tab alongside "All" and "Ad reviews"
- **All view**: merges review + schedule items, sorted by `c.order` descending (most recently added first)
- **Schedule cards**: blue left accent (`#1677ff`); copy: *"[Company]'s ad campaign is ready to schedule"*; CTA "Schedule" → `openAdPanel(ref)` (panel already opens in playlist assignment mode for approved campaigns — no new mode needed)
- **Review button**: removed `switchTab('review')` — panel opens directly on Overview page since it's `position:fixed`

---

## SP Portal: Copy / label fixes
- Priority Actions card titles: changed "creative" → "ad campaign" (`needs approval` and `is ready to schedule`)
- Priority Actions + Upcoming Campaigns: changed `c.buyer` → `c.company` (company name more actionable for ops than uploader name)
- Metrics: removed redundant *"1 creative pending"* subtitle under "Creative approvals needed"; replaced with static *"Needs review"*

---

## SP Portal: Upcoming Campaigns
- Removed "View schedule →" button (not actionable from that context)
- Each campaign row now clickable → `openAdPanel(ref)`; subtle hover highlight added

---

## SP Portal: Internal ad upload — playlist assignment redesign
**Design decision**: Replaced "auto-assign to all playlists" model with explicit opt-out selector

### New Playlist modal
- Added "Internal ads" section: checkbox *"Allow SPX internal ads to play on this playlist"*, default ON
- `savePlModal()` now writes `rules.int` from this checkbox

### Upload Internal Ad modal
- Replaced raw checklist with **collapse/expand pattern**:
  - Default (collapsed): *"All N playlists selected"* + *"Customize →"* button — zero scrolling for common case
  - Expanded: full checklist, all pre-checked for playlists where `rules.int: true`; ops can uncheck any
  - Summary text live-updates to *"X of N playlists selected"* when customized
- `openUploadModal()` renders checklist dynamically so new playlists appear automatically
- Info banner updated: *"Internal ads skip the review queue. Select which playlists this ad should play on."*
