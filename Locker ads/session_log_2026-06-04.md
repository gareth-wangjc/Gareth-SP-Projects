# Session Log — 2026-06-04

## Files touched
- `SPX_ops_locker_ads_management_24.html` (SP Portal)
- `NSS_seller_place_SPX_Locker_ads_v3.html` (NSS Seller)
- `Shopee_seller_place_SPX_Locker_ads_v4.html` (Shopee Seller)

---

## SP Portal — Ad Management tab

### Table
- Replaced "Review date" column with **Campaign period** (`YYYY-MM-DD to YYYY-MM-DD`)
- Added **Campaign ID** column (new format: `SPXSGAD` + 2-digit year + 12 digits, e.g. `SPXSGAD260000000390`)
- Added **Exposure type** column ("By Spots · 9,000" or "By Duration") — previously buried in sub-line
- Ad name cell is now clean (no sub-line)
- Status column: removed colour pills, plain text only
- Action column: **"Review"** (red button) for pending review ads; **"View"** (blue text link) for all others
- Table sorted by most recently submitted first
- Fixed nonsensical "25–55 Apr" for FairPrice Q2 — proper cross-month dates throughout
- All dates standardised to `YYYY-MM-DD` format (aligned with logistics processes)
- Campaign IDs updated across all CAMPS data to new format

### Search filters
- Added **Campaign ID** search input
- Added **Exposure type** dropdown filter
- Reset button clears all five filter fields

### Ad review panel (sidebar)
- Removed coloured status pills from top; status and ad type moved as plain text rows under Booked scope
- **Two-step approval flow** implemented:
  - **Step 1** (creative review): "Request changes" + "Approve" (pending) or "Next →" (already approved)
  - **Step 2** (playlist assignment): "Cancel" + "← Back" + "Assign playlist"
- "Request changes" disabled until ≥15 characters in feedback field; live countdown hint
- **"Request changes"** → sets `rejected: true`, clears approvedDate, status = **Rejected**
- **"Approve"** → sets approvedDate, status = **Approved**, moves to Step 2
- **"Assign playlist"** → saves playlists, checks startDate vs today → **Scheduled** (future) or **Live** (past)
- **"← Back"** → keeps Approved status, forces panel back to Step 1 with "Next →" CTA
- Pre-approved campaigns (approvedDate set, no playlists) open directly on Step 2
- FairPrice Q2 set to Approved state (future start 2026-09-01) as edge case demo
- Panasonic TOUGHBOOK set to future start (2026-07-01) to demonstrate Scheduled status on assignment
- AI screening flagged items: changed from orange → **red** (per BRD: Pass ≥95% green, else red)
- Review history redesigned to match Order Tracking History UI style: stacked timestamp left, hollow dot + line, event + operator right; no colour coding
- Operator in review history now shows actual contact name (e.g. "K. Watanabe") instead of "Seller"
- "← Back" added to playlist assignment footer

### Overview tab
- Removed "Invoices overdue" metric; metrics grid changed from 4-column → 3-column (Creative approvals, Active campaigns, MTD revenue)
- Removed "Invoices" filter button from Priority Actions; only "All" and "Ad reviews" remain
- Removed all invoice/finance data: FIN_LOG, overdueInvoices, solvency fields, payment rows

### Playlist management
- New playlist modal: added info hint reminding ops to include all location types and regions to avoid lockers receiving no seller campaigns

---

## All portals — Status rename
- **"Action required" → "Rejected"** across SP Portal, NSS Seller, and Shopee Seller
- Affects: status filter dropdowns, table row data-status attributes, status chips, panel header text, JS filter functions

---

## Seller portals (NSS + Shopee) — Create Ad page
- Removed spec tags (1080×1920px, JPG/PNG, etc.) — redundant with description line
- Replaced with **"What SPX reviews"** grey box: 7 industry-standard content rules (no competitor logos, factual claims, age-appropriate, safe zone, font size, QR codes, no strobing)
- Removed dropdown + "View All ⤢" from Locker Preview panel
- Removed "Each ad slot plays for 10 seconds" from preview notes
- White background applied throughout

---

## BRD discussion (section 2.1)
- Aligned section title from OOH → **DOOH**
- Discussed and corrected DOOH vs OOH industry practices:
  - Screen-level filtering (locker-level targeting) is industry standard in DOOH, not OOH-specific
  - Playlist matching uses intersection logic (≥1 locker match = assign), not exact/strict subset
  - AND logic for multi-criteria ads: all criteria must match a single screen
  - Playlist still needed for: rotation sequencing, content policy, fallback content, operational scale
- Corrected flow diagram: seller sends targeting criteria → DOOH matches to playlists (not "same" criteria but "overlapping")
- Flagged and corrected: "DOOH advertisers" → "DOOH platform providers" (Yodeck, ScreenCloud)

---

## Key decisions made today

| Decision | Detail |
|---|---|
| Campaign ID format | `SPXSGAD` + 2-digit year + 12 digits = 19 chars total (not 14 as initially counted) |
| Date format | `YYYY-MM-DD` aligned with logistics |
| Approval flow | 2-step: Approve (creative) → Assign playlist. Statuses: Pending review → Approved → Scheduled/Live |
| Rejected status | Replaces "Action required" everywhere; creative can be resubmitted |
| Locker guardrails | No hard blocks on locker→playlist assignment; soft reminder in modal; monitoring recommended as future requirement |
| Playlist hint | "Include every location type and region where this playlist's lockers are placed. Any locker not covered won't receive seller campaigns." |
