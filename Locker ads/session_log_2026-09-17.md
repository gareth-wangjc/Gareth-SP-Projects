# Session Log — 2026-09-17

## Context
Continuation of the Phase 2 PRD/Finance work from 2026-09-16. This session covered: reworking the Finance Portal rate card structure per Finance PM feedback on Flat Fee vs Tiered, rebuilding the Mass Create GSheet template to match, researching the SG e-invoice (SBC+EBS) PRD to answer whether Locker Ads payment needs to integrate with it, rewriting §3.3.2's cost formula for the new rate structure, a full read-through review of the live Phase 2 PRD, and — the most consequential correction — discovering the Flat Fee/Tiered labels had been assigned backwards for most of the session and swapping them everywhere.

## 1. Rate card structure rebuilt: grid+discount-ladder → unified Rate Bands table (`finance_portal_ads_rate.html`)

Finance PMs said the previous grid (one rate per Location Type × Region cell) + separate Coverage%→Discount% Fee Table didn't match how Flat/Tiered actually needs to work.

- **New shape**: one row-based table per rate card version — Location Type (dropdown) → Region (cascading dropdown, filtered to combos enabled in SP Portal Booking Configuration) → Coverage Start % → Coverage End % → Amount (SGD per locker, no "per day"/"per week" — Gareth's call, the ×7 lives in the billing formula instead, not the rate card).
- **Two validation shapes depending on Fee Structure** (see §7 below for the final correct label mapping):
  - Contiguous-bracket structure: bands must be contiguous, no gaps/overlaps, span 0–100% per Location Type × Region. Billing sums each bracket's portion (marginal/progressive) — "beyond a threshold, lockers from there on are charged a different rate."
  - Nested/single-lookup structure: bands must all start at 0%, no duplicate End %, must reach 100%. Billing applies the single smallest-matching band's Amount to the entire quantity — "the whole qty within the campaign is charged one rate."
- Every enabled Location Type × Region combo must have ≥1 band or the step is blocked ("Missing a rate for: ...").
- Rewrote seed data (RC-0001/0002/0003) to the new shape, each demoing one combo with real multi-band examples.
- Removed dead grid/fee-table CSS and JS (`renderGrid`, `gridComplete`, `syncGridFromDOM`, `renderFeeTable`, `validateFeeTable`, etc.) entirely rather than leaving it unused.
- Cross-file cleanup: `SP_portal_ads_management.html`'s Finance-callout banner and `NSS_place_locker_ads_shipment_portal.html`'s `AD_RATES` comment updated to drop "per day"/"per week" wording. Flagged (not fixed) that NSS's own half/full-keyed `AD_RATES` hasn't been reconciled to this new band shape — still a known gap.
- Verified via headless Playwright (cached copy under `%LOCALAPPDATA%\npm-cache\_npx\e41f203b7505f1fb\node_modules`): validation blocks on missing combos, View/Edit-draft pages render/load the new band shape correctly, Location Type → Region cascade filters correctly (e.g. Condominiums excludes OCR) — zero JS errors.

## 2. Finance Portal tab renamed "Locker Advertisement" → "Locker Ads Fee"

Gareth's own PRD-comment reminder ("change to smth cost related in meaning") — settled on "Locker Ads Fee" to match "SP Commission"'s naming convention (named after the fee itself, not framed as Cost/Revenue, since the tab also holds GST which isn't revenue). Updated the tab label, both page breadcrumbs, page `<title>`, and code comments in `finance_portal_ads_rate.html`, plus the cross-reference in `SP_portal_ads_management.html`'s banner text.

## 3. Mass Create GSheet template redesigned to match (external Google Sheet, not a repo file)

Iterated through several rounds with Gareth directly editing the sheet and me reviewing:
- Columns: Seller Type / Merchant Type / Fee Structure / Location Type / Region / Coverage Start % / Coverage End % / Rate per Locker / Effective Start — no more Row Type discriminator (Rate vs Discount Tier), since the new model has one row shape only.
- Caught and corrected: example rows that split one Location Type × Region's bracket set across two Regions (invalid — every band in one set must share both Location Type and Region), and a single-band row that didn't span through 100% (invalid for the contiguous structure).
- Decided **against** "blank Rate = $0" — blank must reject the upload; $0 must be typed explicitly if genuinely free, so a data-entry mistake never silently becomes a free ad slot.

## 4. E-invoice (SBC+EBS) PRD reviewed — concluded it doesn't apply to ads payment

Gareth asked whether Locker Ads billing needs to integrate with SG's e-invoice PRD (`E-invoice Generation for NSS`, pageId 3144639411), since Evelyn said it's complicated (SIS vs SBC+EBS varies by market).

- That PRD's SBC+EBS flow is a **cycle-end, bill-then-chase mechanism** — it only fires when Sender Balance auto-deduction *fails* at the end of a seller's billing cycle (a batch export → SeaTalk bot → manual SBC upload → EBS generates the invoice → seller acknowledges → uploads payment proof → ops confirms settled). It's an accounts-receivable document, not a receipt.
- Locker Ads pays upfront via Sender Balance/QR *before* scheduling — no debt state ever exists, so this pipeline structurally doesn't apply. The real thread to pull was "what does the successful-Sender-Balance-payment path already do for invoicing" (that PRD explicitly defers this as pre-existing/out of scope).
- **Resolved separately**: Gareth confirmed with SG/NSS ~1h before this session that there's currently no receipt for shipping fee either, so ads inherit "no receipt yet, revisit as a separate request if needed" — the apparent contradiction with "SG says it's a legal requirement" was a timing issue (stale info on my side), not an unresolved doc conflict.

## 5. §3.3.2 (cost formula) rewritten for the new band structure

Replaced the old "base rate + separate volume-discount %" model with a formula keyed directly off the rate bands:
- Contiguous-bracket structure: `Combination Cost = Σ(band_billed_lockers × band's Amount across every band touched) × 7 × weeks`, walking bands from 0% and stopping once a band's Start % exceeds Coverage %.
- Nested/single-lookup structure: `Combination Cost = billed lockers × (smallest matching band's Amount) × 7 × weeks`, no splitting.
- No separate discount-% multiplier at the end anymore — discount-as-coverage-grows is entirely expressed via which band(s) get read.
- 3 worked examples drafted and handed to Gareth to paste in (single-combo nested-lookup case, single-combo bracket-split case with a rounding example, multi-combo sum case).

## 6. Full read-through review of the live Phase 2 PRD (pageId 3295849790)

Gareth asked for a soundness check, especially Finance. Findings, by severity:

**Critical**: §3.1.2's rate-band auto-fill description had Flat Fee/Tiered's behaviors swapped (fixed in §7 below); §3.6.3's Shipment Portal tables still had "Total Amount"/"Total Collected" as two separate columns instead of the already-locked single-column collapse (Gareth fixed this to "Amount Collected", removed "Total Amount"); §3.6.5 (refund) has no pro-rata amount formula, only describes the API call chain (Gareth: leave for now); §3.6.4's "no receipt yet" note — resolved, see §4 above (not a bug, just stale on my end).

**Medium**: §3.4.3's locker-allocation logic was hardcoded to exactly 2 packages ("divide by 2" for half) — contradicts §3.1.3's own scalable-package goal; Gareth generalized the wording to "based on the % configured as a package to them" (still has one residual gap flagged: the worked example's ">5/<10"/"<5" bullets still don't cover exactly-5-available). §3.1.2's Action column ("view, disable, copy") — Gareth fixed to match the real portal. §3.5.2 conflating "Approved" status with payment eligibility — Gareth confirmed the sequence (Approved → Pending Payment → Scheduled) is intentional/correct as informal section framing; the stray "Status: Live" field in that section wasn't confirmed fixed. §3.5.4 (seller edit, missing repricing mention) — Gareth removed the whole section, since an upcoming multi-reviewer approval requirement supersedes it.

**Low**: duplicate/truncated section headers (§3.4.3.2, a second truncated §3.4.4) — Gareth removed. Two open questions left inline in the doc text (§3.6.2 "how to show refund transactions", §3.6.3 "is this where refund tagging shows") — left as-is, still open. §3.1.2's "overwrites all older rate cards" wording — Gareth confirmed this is correct as written, already clarified with the finance team directly.

## 7. Flat Fee / Tiered labels were backwards — corrected everywhere (significant)

After locking the whole rate-band redesign (§1–§6 above) under one label mapping, Gareth corrected the mapping twice, landing on:
- **Tiered = contiguous brackets** (marginal/progressive — "beyond a threshold, lockers from there on get a different rate")
- **Flat Fee = nested-from-0, single-lookup** (whole quantity billed at one rate — "the whole qty within the campaign is charged one rate")

This is the reverse of what was built and verified in §1–§5. Swapped everywhere:
- `finance_portal_ads_rate.html`: `validateBands()`'s two branches, `updateBandsHint()`'s two hint strings, the View page's hint text, the header data-model comment, and which seed rate card version (RC-0001/RC-0002) is labeled which structure. Re-verified via Playwright after the swap — zero JS errors, correct hint text per structure.
- `SP_portal_ads_management.html`'s Finance-callout banner text.
- Gave Gareth the exact corrected text for §3.3.2 (step 3's two formula branches, all three worked-example labels) and the Mass Create GSheet (row 3's Fee Structure guiding text, and which example card rows need their Fee Structure value flipped) to paste in himself. He applied both to the PRD; two small residual leftovers were caught (Example A/B's inline "Tiered bands:"/"Flat Fee bands:" descriptor line hadn't been swapped along with the example title) and corrected.

**Root cause note for future sessions**: the confusion arose because early in the session Gareth confirmed my proposed mapping directly (agreeing nested bands should resolve to "the smallest matching band" — which is objectively correct mechanically), but the *label* attached to that mechanic (which of "Flat Fee"/"Tiered" it should be called) wasn't independently double-checked at that moment, and got assigned by me based on assumption rather than confirmation. Worth explicitly confirming label-to-mechanic mapping in words, separately from confirming the mechanic itself, next time a similar two-structure naming decision comes up.

## Verification approach this session
All `finance_portal_ads_rate.html` changes checked via headless Playwright (same cached copy as prior sessions, `NODE_PATH` pointed at `%LOCALAPPDATA%\npm-cache\_npx\e41f203b7505f1fb\node_modules`) — zero JS errors confirmed after both the initial rebuild and the later label swap.

## Open items / not yet built
- NSS's `AD_RATES` (half/full-keyed flat rates) still not reconciled to the new Location Type × Region × Coverage-band shape — flagged twice now, not yet scheduled.
- §3.4.3's exactly-5-available boundary gap in the worked example (not the general rule, which is now correctly generalized).
- §3.6.5 pro-rata refund formula still missing from the PRD (Gareth: leave for now).
- Two open questions still unanswered in §3.6.2/§3.6.3 about how refund transactions display.
- §3.5.2's "Status: Live" stray field — not confirmed fixed.
- Mass Create .xlsx/GSheet still needs the Flat/Tiered label swap's knock-on wording applied by Gareth (given as copy-paste text this session; not verified as applied).
