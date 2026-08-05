---
name: project_add_to_circle_feedback
description: "Add-to-circle now toasts on success/failure and persists its ✓ via a new server ?contains flag. Web live; mobile #2 staged but NOT built."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Fixed two tester-reported gaps in the "Add to circle" control (the thing that
replaces follow — adding to a circle IS the follow):

1. **Feedback (web).** `AddToCircleButton.jsx` `handleAdd` used an empty
   `catch {}` (errors on the floor) and only swapped a tiny icon on success.
   Now it fires a `toast.success` ("Added X to Following") with an `action`
   link `to:/circles/<id>`, and `toast.error` on failure. (`toast` from
   `frontend/src/app/toast.js`; API: `toast.success(msg, { detail, action:{label,to} })`.)
2. **Persistent ✓ (web + mobile).** The added-state was ephemeral local state,
   so the checkmark reset on reload/navigation. Now seeded from real membership.

**Server primitive (`GET /users/:id/circles?contains=<memberId>`):** owner-only.
When `contains` is passed, each returned circle gets a boolean `contains` flag
(one indexed query over `members.id`). Added to `routes/users/circles.js`;
plumbed through `@kowloon/client` `feeds.getUserCircles({ userId, contains, limit })`.
Note that route only returns `type:"Circle"` circles — but **Following is
type Circle**, so it IS included (see [[project_following_circle_gotcha]]).

**Deploy state (2026-08-04):**
- Web: BOTH fixes LIVE on kwln.social + kwln.city (server+client+frontend
  rebuilt/deployed; 4 repos touched: kowloon, kowloon-client, kowloon-frontend).
- Mobile (`ProfileActions.jsx`): only the #2 seed applied (mobile has no toast
  system; its picker modal shows the ✓ inline + Alerts on failure, so it didn't
  need #1). **Shipped in the 2026-08-05 tester build.**

Related: [[project_tester_feedback_open_items]], [[circle_ux]], [[web_app_parity]].
