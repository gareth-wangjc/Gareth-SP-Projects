# Branch Recommendation Logic: SLS emap Supporting PRD

Supporting PRD that makes the buyer checkout emap work with MPL's revamped branch recommendation on the Shipping Options Page (SOP).

## The idea
- **MPL owns recommendation on SOP.** Slot 1 = last used branch (else nearest available used). Slot 2 = nearest never-used branch with the Recommended tag.
- **The emap is a map, so it sorts by distance.** It does not follow SOP rank.
- **MPL passes two things into the emap:** the reference point (last used branch, else buyer address) and the recommended branch ID, when one applies.
- **First load only:** last used on top if selectable. If not, the recommended branch is pinned at the top with the Recommended tag and becomes the reference point.
- **After search or pan:** existing logic. Distance from the new point, no pin, no tag.

## Why this design
- **Following SOP rank breaks the map.** Slot 2 can sit far from Slot 1, so tapping through cards makes the map jump between areas.
- **No ranking API in MPL, no ranking rules in SLS.** The 7 Oct `rank_branches` design was dropped on 20 Oct.

## Current status
- **Phase:** PRD v6 updated on Confluence (21 Oct). Update email sent to markets on 21 Oct.
- **Goal:** keep markets signed off on the changed PRD, then hand to devs.
- **Timeline:** no new sign-off date. Earlier 15 Oct sign-offs stand unless a market objects. Gareth reviews with devs while local product reviews with local biz.
- **Open:**
  - Kai Yang to confirm launch param effort (reference point and recommended branch ID).
  - What MPL passes for the A/B Control group (draft assumes nothing).
  - Rule for buyers with no last used branch.
  - Whether MPL sends the recommended branch as the reference point when last used is not selectable.
  - SLS dev to confirm the pin works on top of the existing grouping.
  - PD effort to align the Recommended tag UI (incl. PC) with SOP.

## Who does what
- **Gareth:** PRD owner, market comms, dev review.
- **Kai Yang (MPL):** Master PRD owner, launch params, A/B bucket.
- **SLS dev:** emap sort, pin and tag.

## Key documents
| File | What it is |
|---|---|
| [PRD_signoff_email.html](PRD_signoff_email.html) | 15 Oct sign-off email (old rank_branches design) |
| [PRD_Update_email.html](PRD_Update_email.html) | 21 Oct update email draft. Gareth sent a reworded version |
| [progress-2026-10-07-api-contract.md](progress-2026-10-07-api-contract.md) | SUPERSEDED. `rank_branches` contract and display groups (3 groups still hold) |
| [progress-2026-10-06-emap-prd-planning.md](progress-2026-10-06-emap-prd-planning.md) | SUPERSEDED. Sources, today's SOP vs emap behaviour, MPL's SOP model |
| [discussion-slot-logic-2026-07-13.md](discussion-slot-logic-2026-07-13.md) | SUPERSEDED. July slot-anchored model |

Links:
- PRD: [SSCP page 3326543451](https://confluence.shopee.io/pages/viewpage.action?pageId=3326543451)
- Master PRD: [SPCT page 3166396134](https://confluence.shopee.io/pages/viewpage.action?pageId=3166396134)
- POP ticket: [SPCPM-113206](https://pop.shopee.io/request-management/view?issue_key=SPCPM-113206)

## Daily log
One file per working day in [daily-logs/](daily-logs/). Newest first.

| Date | Summary |
|---|---|
| [2026-10-21](daily-logs/2026-10-21.md) | Dropped rank_branches. Emap sorts by distance, MPL passes reference point and recommended branch. PRD v6 and update email done |
