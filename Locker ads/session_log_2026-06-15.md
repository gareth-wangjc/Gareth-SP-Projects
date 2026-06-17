# Session Log — 2026-06-15

## Files touched
- `SP_portal_ads_management.html` (SP Portal)
- `NSS_place_locker_ads_shipment_portal.html` (NSS Seller)
- `Shopee_place_locker_ads_seller_centre.html` (Shopee Seller)

---

## SP Portal — Edit button scoped to Scheduled only

**Change:** Edit button in the ad panel footer now only appears when `c.type==='int' && c.status==='Scheduled'`. Previously it showed for both Scheduled and Live internal ads.

**Reason:** Industry standard (DOOH platforms: Broadsign, Vistar, Clear Channel) — once a campaign is live it is actively playing on physical screens; editing mid-flight requires cache invalidation and screen pushes. The correct flow for fixing a live campaign is cancel → re-upload.

**Behaviour after change:**

| Campaign state | Footer buttons |
|---|---|
| Scheduled internal ad | Close · Edit · Cancel campaign |
| Live internal ad | Close · Cancel campaign |
| Scheduled seller ad | Close · Cancel campaign |
| Live seller ad | Close |

---

## All portals — Edit history: per-field entries → single "Edited information" with before/after block

**Change:** Previously each changed field pushed its own log entry ("Edited campaign name", "Edited run period", etc.). Now all changes from a single save are collected into one `{event: 'Edited information', changes: [...]}` entry with a before/after block.

**Reason:** Per-field entries were misleading — if two fields were changed in one save, it looked like two separate events. A single grouped entry with before/after clearly shows what changed atomically.

**Format in tracking history:**
```
Edited information
operator: gareth.wangjc
┌─────────────────────────┐
│ Before                  │
│ Name: SPX Brand May     │
│ End date: 2026-05-31    │
│ After                   │
│ Name: SPX Brand Mayee   │
│ End date: 2026-05-30    │
└─────────────────────────┘
```

**NSS + Shopee seller saves:** Still push a separate "Pending review" entry immediately after "Edited information" — two distinct events since they are logically different (edit vs. status change).

**Functions updated:**
- `saveInternalAdEdit(ref)` — SP Portal
- `saveNSSEdit(id)` — NSS
- `saveSHEdit(id)` — Shopee
- `renderReviewLog(log, sellerView, buyerName)` — SP Portal + Shopee
- `renderNSSReviewLog(log, sellerView)` — NSS

---

## All portals — "Run period" → "Campaign period"

**Change:** All instances of the label "Run period" renamed to "Campaign period" across all three portals — edit form field labels and the `field` key in the before/after change log.

**Locations changed (5 total):**
- SP Portal: upload internal ad modal label + edit form label
- NSS: edit form label + `changes.push` field name
- Shopee: edit form label + `changes.push` field name

---

## Design decisions made today

| Decision | Detail |
|---|---|
| Edit = Scheduled only (not Live) | DOOH standard — live ads are on physical screens; editing mid-flight requires cache invalidation. Cancel → re-upload is the correct fix flow. |
| Grouped before/after edit entry | One save = one history entry. Prevents misleading multi-line log when multiple fields changed simultaneously. |
| Separate "Pending review" entry | Status change is a distinct event from the edit itself — keeps audit trail clean. |
| "Campaign period" label | Consistent with existing panel field labels; "Run period" was non-standard. |
