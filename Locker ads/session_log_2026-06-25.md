# Session Log — 2026-06-25

## Status: No mockup changes made this session

---

## SG Feedback Received

**Feedback:** SG wants a seller-configurable billing cap for By Duration campaigns — set at campaign creation. If actual spend exceeds the cap, the ad stops displaying.

> "Put a limit at the point of campaign duration --> seller to configure. If exceed then don't display anymore"

---

## Brainstorm completed (hold for implementation)

Two related changes are needed in the seller creation flow (both NSS and Shopee portals):

1. **Spending cap field** — optional input shown below the Est. Cost row in Section 2 (Ad Exposure), visible only when "By Duration" is selected. Warns seller if cap is set below the estimated cost.

2. **Spot price per location type** — SG has indicated they may also want pricing to vary by location type (not a flat SGD 0.15/spot). Spec not yet confirmed.

**Decision: Hold both changes and implement in one go once SG finalises the pricing-per-location-type spec.** Both touch the same UI section (Section 2 duration block) and the BRD billing table, so batching avoids double-rework.

---

## Design notes for when SG confirms

- Cap field placement: below "Est. cost" row in the By Duration block, labelled "Spending cap (optional)"
- Soft warning (not a blocker) if cap < estimated cost: "Cap is below your estimated spend. Your ad may stop early."
- Runtime: ad stops at next playlist refresh cycle after cap is crossed (not mid-slot)
- New campaign status: **Cap reached** (distinct from Completed and Cancelled)
- SP Portal sidebar: show cap amount alongside actuals once campaign is live
- BRD billing table: replace "There is no billing cap" remark with cap behaviour description
- Cap editable by seller while campaign is pending review only; locked once approved/running
