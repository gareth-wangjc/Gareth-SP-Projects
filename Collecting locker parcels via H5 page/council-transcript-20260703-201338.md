# LLM Council Transcript — BRD Logic Gaps & TW Collect-Screen Design

**Date:** 2026-07-03
**BRD:** "Collecting locker parcels via H5 page" (Shopee SPX Inhouse Lockers), Google Doc `1M7x77SbEEFTTdE-hAFFxjIHWHyfdHrXn_XWvHFIQPSQ`

## Original Question

Identify logic gaps in the BRD, and scrutinize the TW design for "seeing where to collect which parcels" (mockup excluded for now — TW shows only cell ID + TN, no visual locker layout).

## Framed Context (given to advisors)

Buyers scan a locker QR → phone becomes a token key to open cells, avoiding the shared locker screen (drivers occupy it ~10 min). Phase 1. Key TW facts: uses a third-party payment-machine vendor (no SPX locker screen); toggles OFF phone-token setup (PIN every visit); toggles OFF geofencing (blocks remote collection); blocks COD parcels from the H5 (must pay + collect ALL at the machine). TW's signoff row in the BRD is blank. Full BRD sections 3.1–3.9, 4.0 metrics, 6.0 signoff were provided.

---

## Round 1: Advisor Responses

### The Contrarian
Logic gaps: Numbering collision — two sections labeled 3.2.1. Ordering contradiction: remote detection fires "on scan" but 3.2.2 says session token generated on scan required for all actions — which runs first? Does a recurring token-holder bypass the geofence check? 3.1.1 driver conflict under-specified: if "Cell Busy" for A01, do the buyer's OTHER cells open? 3.5 fifteen-minute windows conflict: PIN valid 15 min, cells released after 15 min, but session token is 10 min — three clocks, no anchor time. 3.3.2 vs 3.4 COD collision: nothing forbids the non-TW payment flow rendering if the TW toggle is misconfigured. Missing branches: partial payment failure; report-issue while a door is open; TW has no signoff yet carries the most custom logic. TW collect design: stripping the visual layout is the real defect — the highest mis-collection-risk market gets the weakest UI; it contradicts the BRD's own "many parcels in 1 locker, check before closing" risk.

### The First Principles Thinker
Purpose splits in two: (a) skip the driver-occupied screen; (b) help buyers find cells at wide sites. TW uses a third-party machine, PIN every visit, blocks remote, and forces cash-COD buyers to the machine to pay AND collect ALL — so for many TW buyers the H5 delivers nothing the machine doesn't; it relocates TW's constraint, doesn't remove it. The blank TW signoff is the tell; reframe TW as its own decision, not a toggle set. Gaps: (1) PIN-vs-token contradiction — 3.2.1 rejects PIN-as-OTP because shared helper PINs mint permanent tokens (theft), but TH demands PIN precisely because buyers use helpers; delegation is undecided (feature or threat?). (2) Timer mismatch 10 vs 15 min. (3) Remote opening breaks the "you scanned on-site so you're present" security premise. (4) Offline paradox: if the locker internet is down, the H5 (needs BE) can't even render "go use the screen."

### The Expansionist
Metrics (4.0) are a scoreboard not a growth engine — you don't track token opt-in rate or repeat-token collection rate, the most valuable number; every token user becomes a zero-SMS, re-identifiable cross-locker user = a loyalty ID. Token is under-monetized identity — a persistent key at "any SPX Locker" is a cross-locker user graph; enable pre-authenticated arrival, gamification firing only for token users to drive opt-in. Remote-open is logged as an ops flag, not a product — this is the helper economy TH flagged; formalize delegated collection (scoped one-parcel token to a helper). No multi-locker wayfinding. TW design: the cell-ID + TN list is a downgrade disguised as market-fit; cash-only doesn't require dropping the visual layout (payment at the machine, collection still physical door-finding); keep the diagram, add highlight-in-sequence; the blank TW signoff is permission to pilot the layout, not strip it.

### The Outsider
"Collection PIN" (6-digit) vs "phone verification" vs "4-digit one-time code" = three number-things in one flow; a customer who got a 6-digit text then sees a 4-digit ask assumes an error. The T&C line "this phone can open your lockers without a PIN" is alarming — reads as "a thief opens my parcels if I lose my phone"; no un-link/revoke mentioned = kills adoption; "your lockers" plural is confusing when I have one parcel. The 50m geofence popup is backwards — if I'm standing there and GPS drifts (common indoors/by lockers) the app accuses me of lying and offers a scary "remote" option I never asked for. "COD / tracking number / cell ID" is jargon; lead with what the item is or the shop name. The 10/15-min rules are invisible until tripped — customers panic at "expired." TW wall-of-lockers with a text list: it fails — cell IDs mean nothing without a map; at minimum the opened doors should physically pop/light up.

### The Executor
3.2 vs 3.5 timing contradiction — 10-min session vs 15-min re-scan/PIN; does a re-scan mint a fresh token or does the 10-min token block the reopen at min 10–15? 3.2 has no OTP/PIN attempt limit — the PIN path has no lockout and no lockout scope (per-session? per-phone? per-locker?) = an unbuildable security criterion. 3.1.1 "Cell Busy" race — who owns conflict resolution, firmware or backend; partial-success state undefined. Duplicate 3.2.1 + garbled geofence sentence; GPS-vs-WiFi-IP disagreement has no tiebreak. 3.4/3.6 have no failure states — no payment timeout/failed branch, no handling if Device Dashboard is down. TW collect screen: buildable but risky; no visual layout = high error for multi-cell; worse, TW COD forces "pay AND collect ALL at the machine" but 3.5's 15-min reopen is described only for the visual variant, so a TW buyer who leaves a parcel has NO defined recovery; the TW QR comes from a third-party vendor machine with no API contract/error format/offline fallback (3.9 assumes an SPX screen TW doesn't have) — an ownership gap.

---

## Round 2: Peer Reviews (anonymized A–E; mapping: A=Contrarian, B=First Principles, C=Expansionist, D=Outsider, E=Executor)

**Reviewer 1:** Strongest: E — only one treating the BRD as buildable-or-not; sharpest TW insight (15-min reopen only defined for visual variant → TW COD buyer has no recovery; vendor has no API contract). Biggest blind spot: C — monetizing an identity graph while ignoring buildability/safety; reframes the shared-PIN theft vector as a feature. All missed: who is accountable when the wrong parcel is collected (liability/dispute/refund ownership); accessibility; the buyer-isn't-the-collector delegation reality.

**Reviewer 2:** Strongest: E — treats each gap as a buildability test (no PIN lockout scope, Cell Busy ownership, TW no-recovery + no vendor API). B close second for reframing TW as a decision. Biggest blind spot: C — growth-strategy fan-fiction on an unresolved spec. All missed: whether the H5 adds enough value over the existing screen/machine to justify the new attack surface; accessibility/no-smartphone; liability for mis-collection.

**Reviewer 3:** Strongest: E — flags unimplementable-as-written criteria; uniquely lands the TW ownership gap (3.9 assumes an SPX screen TW doesn't have). Biggest blind spot: C — building a loyalty ID on a spec that can't define its own lockout. All missed: buyers with no smartphone/data at the locker (the exact COD population); accessibility/language; liability when a remote-open exposes the wrong parcel.

**Reviewer 4:** Strongest: E — build-blocking altitude (unbuildable lockout scope, Cell Busy ownership, decisive TW third-party-machine API/offline gap). B close second (H5 relocates TW's constraint). Biggest blind spot: C — monetizing an identity layer that isn't shippable; celebrates the cross-locker graph without noticing it's a privacy/consent liability. All missed: legal/privacy consent (PDPA/GDPR) for persistent phone-to-locker tracking; baseline mis-collection & SMS-cost numbers (without which 4.0 metrics prove nothing).

**Reviewer 5:** Strongest: B — the only response that reframes rather than itemizes (H5 relocates TW's constraint; PIN-vs-token contradiction reveals an undecided delegation policy = the root question). E runner-up, most rigorous on buildability. Biggest blind spot: C — treats an unfinished, self-contradicting BRD as a monetization canvas. All missed: no-smartphone/no-app fallback; privacy/consent legal basis for a persistent cross-locker token.

---

## Chairman Synthesis

### Where the Council Agrees
Five advisors converged on a hard core of defects — treat as fixes, not opinions:
- **Timer system is incoherent.** 3.2.2 session token = 10 min collides with 3.5 PIN validity = 15 min and cell-release = 15 min. Three clocks, no anchor time, no definition of what a re-scan does. Spec-level contradiction.
- **Duplicate section 3.2.1** and a garbled geofence sentence.
- **Identifier soup** — "Collection PIN" (6-digit) vs "phone verification" vs "4-digit OTP" vs "session token." A buyer who gets a 6-digit SMS then a 4-digit prompt assumes it's broken.
- **"Cell Busy" driver-conflict race (3.1.1) has no owner** — firmware vs backend undefined; partial-success undefined.
- **Security criteria unbuildable** — no PIN attempt limit, no lockout, no lockout scope (3.2).
- **Stripping TW's visual layout is a defect, not localization.** Highest-error market gets the weakest UI; contradicts the BRD's own "check all parcels before closing." Payment-at-machine and the compartment diagram are orthogonal.

### Where the Council Clashes
- **TW: toggle set or its own product?** First Principles & Expansionist: blank signoff = tell; H5 relocates TW's constraint rather than removing it. Executor: buildable-but-risky, missing vendor API contract. Verdict: both, stacked — TW needs a product decision AND a vendor integration spec.
- **Delegation (shared helper PIN): threat or feature?** 3.2.1 rejects PIN-as-OTP (theft) but TW/TH demand PIN because buyers use helpers. Policy question is real and must be decided; monetization is premature.
- **What to do with the token.** Expansionist's identity-graph + gamification unanimously flagged as premature — shelve until contradictions close.

### Blind Spots the Council Caught (all five advisors missed)
1. Liability / dispute ownership (wrong parcel collected; remote-open exposure) — no dispute-resolution section.
2. No-smartphone / no-data fallback (incl. offline-locker paradox).
3. Privacy/consent legal basis (PDPA/GDPR) + documented token un-link/revoke path.
4. No baseline data — 4.0 metrics can't prove improvement without today's mis-collection rate + SMS cost.
5. Accessibility / language.

### The Recommendation
Do not add scope. Harden the spec:
1. **Fix contradictions (blocking):** one anchored timer clock w/ defined re-scan; renumber duplicate 3.2.1 + rewrite geofence sentence; one buyer-facing identifier term; define PIN/OTP attempt limits + lockout scope; assign "Cell Busy" race to one owner + specify partial-success.
2. **Write the four missing sections:** dispute/liability; no-data/offline fallback; consent basis + revoke flow; baseline metrics.
3. **Decide delegation policy explicitly** — authorized access or theft vector? Everything hangs off this.
4. **Reframe TW as its own decision:** restore visual layout (keep diagram + highlight-in-sequence); define vendor QR integration (API contract, error format, offline fallback — 3.9 assumes an SPX screen TW lacks); specify TW leave-a-parcel recovery.
5. **Park the Expansionist agenda** until 1–3 are closed.

### The One Thing to Do First
**Resolve the timer model** — collapse the 10-min session token (3.2.2), the 15-min PIN validity, and the 15-min cell-release (3.5) into one anchored clock, and define exactly what a re-scan does. Every advisor independently hit it; it blocks the build; it silently governs TW's missing leave-a-parcel recovery; only the PM can make the call. Fix before circulating the BRD again.
