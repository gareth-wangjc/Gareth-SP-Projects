# DD x TC Integration

[SPX All] Device Dashboard x Ticket Center Integration: DD and TC tickets move onto FMS Case Management, with auto-assignment and two-way sync between SP Portal and FMS. This folder holds the user guide work and rollout comms (PC version only).

## The idea
- **One ticket system.** Device Dashboard (locker technical tickets) and Ticket Center (buyer, seller and operator enquiries) both run on FMS Case Management.
- **Auto-assignment.** Locker errors raise tickets by themselves and the Assignment Rule assigns them to a team member.
- **Two-way sync.** Reassign, Comment and Response work on SP Portal or FMS and show on both.
- **Old tickets.** Tickets created before go-live stay on a separate old-tickets page, reached from a yellow banner.

## Current status
- **Phase:** user guide is live on the SPX site. UAT email drafted, not yet sent.
- **Goal:** local teams complete UAT by 6 Nov 2026 and go live on 18 Nov 2026.
- **Timeline:** UAT 22 Oct to 6 Nov 2026. Go-live 18 Nov 2026 (11.11 code freeze 4 to 12 Nov). Live testing 18 to 28 Nov 2026.
- **Open:**
  - UAT email needs the Rollout Plan table and the Seatalk groupchat link pasted in before sending.
  - Guide nav status is "upcoming". Flip it to "live" after go-live.
  - Cross-project GitLab token skill still to build.

## Who does what
- **Gareth:** PM. Writes the PRD, user guide and stakeholder emails.
- **Dev team:** configures Case Template, Case Type and the SeaTalk bot in FMS.
- **Local teams:** run UAT, then train ops and settle old tickets.

## Key documents
| File | What it is |
|---|---|
| [UAT_email.html](UAT_email.html) | UAT and rollout email, ready to paste into Gmail |
| [guide-preview/](guide-preview/) | Local preview and source of the user guide |
| [User Guide/](User%20Guide/) | The 24 UAT screenshots used in the guide |

Links: [live user guide](https://ssc-user-guides.gitlab-saas-page.shopeemobile.com/spx/?page=device-dashboard-ticket-center-integration) | [PRD](https://confluence.shopee.io/pages/viewpage.action?spaceKey=SSCP&title=%5BAll%5D+Device+Dashboard+x+Ticket+Center+integration) | [UAT test case](https://docs.google.com/spreadsheets/d/1qEgRC6O3EKmYphGJiL6sJDv1IE_dmellNkgPFZ7MwBE/edit?gid=0#gid=0) | [POP SPCPM-111229](https://pop.shopee.io/request-management/view?issue_key=SPCPM-111229)

## Daily log
One file per working day in [daily-logs/](daily-logs/). Newest first.

| Date | Summary |
|---|---|
| [2026-10-22](daily-logs/2026-10-22.md) | Built and published the user guide (MR !25), drafted the UAT email |
