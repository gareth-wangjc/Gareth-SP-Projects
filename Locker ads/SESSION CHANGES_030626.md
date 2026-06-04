# SPX Ops Portal — Session Edits
_3 Jun 2026 | gareth.wangjc@shopee.com_

File edited: `SPX_ops_locker_ads_management_24.html`

---

## Booking Configuration tab

- **Added delete button** to each AI review criterion card — was missing per BRD 3.1.2
- **Grouped settings into sections** with SP Portal-style red left-bar section headers:
  - "Ad playback" — Ad slot duration + Inactivity timeout
  - "Seller booking options" — Spot packages + Seller-selectable location types + Regions for seller targeting
- **Reformatted layout** from card-based to SP Portal form-row style (label-left / control-right / hint-below), matching the Basic Info page style
- **White background** applied to Booking Configuration tab
- **AI criteria modal** — removed "Example (flag condition)" field (redundant given guidelines); renamed "What to check" → "Guidelines for AI to review"

---

## Schedule tab — General

- **Default landing state** — Schedule tab now lands on an empty white panel with guiding text ("Select a playlist on the left…") instead of auto-loading Playlist 1's gantt
- **White background** applied to all Schedule sub-views (gantt, manage playlists, locker coverage, home state)

---

## Schedule tab — Manage playlists

- **Playlist cards collapsible** — each card has a ▼/▶ chevron; clicking the name/chevron toggles the body; defaults to collapsed on entry
- **Renamed button** — "Rename / edit" → "Edit"
- **Delete playlist** — removed blocker that prevented deletion when lockers were assigned; replaced with a confirmation warning: "X lockers will be left without a playlist and won't play any ads until reassigned"

---

## Schedule tab — Locker coverage

- **Unassigned lockers banner** — removed inline locker list; replaced with a "View more" blue hyperlink that opens a modal showing a table (Locker ID | Locker name | Assign playlist CTA)
- **Removed "Back to schedule" button** from Locker coverage header
- **Assign playlist panel** — removed yellow "No tags added yet" banner; removed "No tags yet" from subtitle; locker subtitle now shows only `ID · Location type`

---

## Playlist logic cleanup (discussed + implemented)

- **Removed "Ad type eligibility"** from the New/Edit playlist modal — redundant given that routing is determined by location types + regions at booking time, not playlist-level ad type rules
- **Updated `campInPl()`** — now uses `c.assignedPlaylists` (explicit assignment) instead of `pl.rules` (type-based proxy); more accurate and consistent with the data model
- **Removed all `rules` references** from UI: gantt info bar, playlist cards, sidebar labels, locker assign/override panels, ad approval panel
