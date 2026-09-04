# Session Log — 2026-09-02

## Context
Gareth is drafting the **Phase 2 PRD** (NS Sellers place locker ads + pay, via SPX Shipment Portal / NSS). This session worked through open logic questions before writing PRD content, using:
- Confluence Phase 1 PRD (pageId 3243446066) and Phase 1.1 PRD (pageId 3272710390)
- BRD Google Doc "Overview" tab (locker allocation logic example: 10 lockers/22 ads example)
- WhatsApp thread w/ Natasha Ho re: billing model conflict (per-spot rate card vs by-duration purchase = bad debt risk)
- Google Slides exec deck "Update on SPX Locker Ads" (rate card, target partners, package options, revenue, rollout plan, ads review flow, payment system backup slide) — file ID `14I5PZc7PGA3RP3DEGafT91ZHdf2WfduOcTQ1b-nQ3M4`
- NSS shipment-order payment flow screenshots (existing "Shipping Fee Payment" modal: QRPH / Sender Balance / Pay at Pickup-Dropoff, plus order-result states incl. auto-refund-to-balance on creation failure)

**Approach agreed with Gareth:** draft PRD section-by-section, terse bullet points / tables / simple flow diagrams only (no prose) — he pastes into the actual PRD himself. Refund logic (open item) explicitly deferred to be discussed **last**, after all other sections are drafted.

## Decisions locked this session

| Topic | Resolved direction |
|---|---|
| Package model | **Half / Full only** — Individual package dropped for this phase (accepting the ~$800/mo sole-proprietor revenue this loses per the exec deck's FY26 projection) |
| Locker allocation logic | Full package requires 100% of matched lockers (by Location Type + Region) available, else greyed out. Half requires ≥50% available → assigns to that %, ranked by availability, tiebreak lowest Point ID. Below 50% available → both greyed out. Nice-to-have: show next-available-period. |
| External ad cap | New FE-configurable field (SP Portal Booking Config) — platform-wide concurrent cap on total external campaigns running anywhere in the network at once (not per-locker/per-package-tier) |
| Billing model | **Flat per-locker-per-week rate card**, tiered by Location Type (Mall/Condo/Office/Others) × Region (CCR/RCR/OCR) × Coverage (Half/Full). No per-spot component — resolves Natasha's bad-debt conflict (by-duration-actual-spots billing can't be known until campaign ends, but seller already pays before scheduling). Rate range $4–$47/locker/week per exec deck. |
| Region taxonomy | Already CCR/RCR/OCR — not a migration blocker (contradicts earlier H5-project memory note that this was still "compass chips"; locker ads specifically already uses CCR/RCR/OCR per the deck) |
| AI review | Confirmed in scope. Two-layer: AI does technical/format check + 1st-round prohibited-content screen; SPX does brand check + 2nd-round content check + landlord-restrictions check; then Layer 1 (SPX) + Layer 2 (management) approval. |
| Management approval | Per-campaign, but folded into the **existing** SPX ops review step already proposed — not a separate manual email gate |
| Payment mechanism | Reuses **existing NSS payment infra** — "Sender Balance" (existing wallet, already used for shipping fees) + market-specific QR gateway (e.g. QRPH for PH). No new wallet/gateway needed. Method set follows whatever that market already has for shipping-fee payment (some markets QR-only, some Sender Balance + QR). Retires the inline-SVG placeholder QR built in the earlier NSS mockup. |
| Refund | **Deferred to end of drafting.** Strong candidate precedent found: existing NSS order-creation-failure flow auto-refunds to Sender Balance — proposed as the model to reuse, but still needs local team confirmation (was Gareth's original open item from 2026-08-25). |
| External enterprise (Singtel, Gov, etc.) | Deferred to a future phase — not in Phase 2 PRD scope |

## Also surfaced (informational, not yet actioned)
- PRD finalization deadline per exec deck's rollout timeline was "End Aug" — already overdue as of today (2026-09-02). Schedule pressure flagged to Gareth.
- Full rollout plan (from deck): rate card mgmt approval End Aug → PRD finalize End Aug → Dev build End Oct → TOS/Legal Mid Sep → Content SOP Mid Sep → NSS trial pitch End Sep → 2-week NSS free trial End Oct → Official NSS launch Mid Nov → Official Shopee Sellers launch Mid Dec → External brands End Dec.
- Ads content policy (prohibited: tobacco/vaping, gambling, adult/explicit, profanity, political, landlord-restricted) and creative template specs (1080×1720px canvas, ~86px safe margin, <10-word headline, 54px min body text) are fully spec'd in the deck backup slides — ready to port into PRD as-is.

## Drafted so far
**§3.1 Booking Configuration** — package/coverage rules, rate card table (partial — Mall + Others rows only; Condo/Office rows still need pulling from the actual rate card), external-ad-cap field, and a booking→rate-lookup flow. One inline open item raised: UX when the external cap is hit (hard block vs waitlist) — not yet discussed.

## Next steps
Continue drafting section-by-section in the same terse bullet/table/diagram format:
- §3.2 NS Seller booking flow (NSS Shipment Portal)
- §3.3 Ad review & approval (SP Portal)
- §3.4 Payment (Sender Balance / QR reuse)
- §3.6 Locker/package allocation (system logic write-up)
- Refund — **last**, once local team has weighed in
