# Session Log — 2026-08-25

## Files touched
- `NSS_place_locker_ads_shipment_portal.html`
- `SP_portal_ads_management.html`

## Context
SG local team aligned on the NS Seller charging/payment flow (discussed 2026-08-21, see PM remarks): after ops approves a campaign, the seller must pay within a backend-configurable window (default 1 working day / 24h) before it can be scheduled. Unpaid campaigns auto-cancel. This session implements that flow end-to-end for NS Sellers only.

**Explicitly out of scope this round** (deferred to a later phase): Shopee seller portal, CCR/RCR/OCR region-taxonomy migration (replacing the current compass Region chips), and the locker/package allocation logic (full/half package assignment).

---

## New status: "Pending payment"
Sits between `Approved` and `Scheduled` in both portals.
- NSS: flat `status` field, same pattern as existing statuses.
- SP Portal: derived via a newly-hoisted shared `adStatus(c)` function (previously duplicated 3x — now one source of truth) — branch added between the `!approvedDate` and `assignedPlaylists.length===0` checks.

## NS Seller Portal (NSS)
- New `#pay-banner` (amber, non-dismissible) above the rejected banner — "N campaign(s) awaiting payment", due-date/time-left subtext, "Pay now" CTA.
- `initBanner()` extended for the new banner; fixed a latent bug where banners were never hidden once their condition cleared.
- New Payment QR modal (`openPaymentModal`): campaign name/amount, locally-generated inline-SVG QR placeholder (no external service), live countdown (`QR_VALIDITY_SECONDS = 180`), expired state with working "Generate new QR" (produces a visibly different pattern), and a "(Demo) Simulate payment received" link → success state → flips campaign to `Scheduled`.
- `PAYMENT_WINDOW_HOURS = 24` — on payment-window timeout, `sweepNSSPayments()` auto-cancels with a distinct reviewLog event (`Cancelled — payment not received`), separate from `Cancelled by seller`.
- Sellers can voluntarily cancel while `Pending payment` (added to `CANCELLABLE`).
- Refactored row-patching logic into shared `updateNSSRow()` (was inline-only in the cancel flow; now also used by payment success and the timeout sweep).
- Two new seed campaigns: **Mid-Year Flash Sale** (`Pending payment`, due in ~19h) and **Weekend Promo — June** (already cancelled for non-payment) — both use `Date.now()`-relative timestamps so the demo reads correctly whenever opened.

## SP Portal (Ops)
- Payment gating: `openAdPanel()` now shows a "Waiting on seller payment" card (red variant if overdue) instead of the playlist-assignment UI until `paidAt` is set — structurally blocks scheduling, not just visually. Hard guard added in `assignPlaylist()` too.
- **No ops-side override** — ops cannot manually mark a campaign paid (confirmed decision); payment can only complete via the seller's NSS QR flow. Ops can only view/wait.
- `approveCreative()` now starts the payment clock (`paymentDueAt`, `amountDue` placeholder) and logs `Approved` + `Pending payment` events.
- Priority Actions: new "Awaiting payment" filter pill + card type; `pendingSchedule` now requires `paidAt` so unpaid campaigns stop appearing as "ready to schedule"; metrics grid 3→4 columns with a new "Awaiting payment" tile (red if any overdue).
- New Ad Settings section "Seller payment": payment window (hours) + QR validity (seconds), same field pattern as existing `est_daily_plays`.
- `sweepUnpaidCampaigns()` runs at `renderTab()` entry and `openAdPanel()` entry — lazy timeout-cancel sweep (moves overdue campaigns to `window.CANCELLED`), since there's no real backend cron in this static prototype.
- Backfilled `paidAt` on the 5 pre-existing approved seed campaigns so the new derivation doesn't regress them to "Pending payment"; internal SPX ads (`type:'int'`) are explicitly excluded from payment gating (`isAwaitingPayment` guards on `type!=='int'`).
- One new seed campaign (Mid-Year Flash Sale, same ref `SGAD000162` as the NSS side) demonstrates the gated/awaiting-payment state; Weekend Promo — June seeded directly into `window.CANCELLED` demonstrating the terminal timeout state.

## Verification
Full click-through flow tested via headless Playwright (banner → QR modal → countdown → expiry/refresh → simulated payment → status flip → reviewLog; SP Portal gating, filter pill, Ad Settings fields, `assignPlaylist` guard, auto-cancel sweep) — all passed, zero console/page errors. Both files' embedded JS also pass a static syntax check.

## Notes / open items for BRD
- `amountDue` is a placeholder display string — no real pricing source yet (pending the deferred package/pricing-tier phase).
- Edge case not resolved: if a campaign is paid late, after its start date has already passed, nothing currently shortens the campaign period or accounts for the lost days — worth a BRD decision.
