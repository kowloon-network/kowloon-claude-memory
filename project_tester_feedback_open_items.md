---
name: project_tester_feedback_open_items
description: "Open items from the first alpha tester (Codswallop) log — phantom profile Share button + follow-payoff legibility; plus the 'web looks like the app' confusion."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

First real alpha tester is **Codswallop** — browser-only (Safari on a Mac,
no Android). His 2026-07-31 log drove the add-to-circle fix
([[project_add_to_circle_feedback]]). Still-open items:

- **Phantom "SHARE button" on user profiles.** He reported a Share button on a
  profile that "isn't connected to an action." There is NO Share button on the
  web user profile — actions are only `Add to` + a `⋯` menu (Mute/Block). Groups
  have Share/Copy-link; profiles don't. Needs clarification from him on where he
  saw it (possibly a post toolbar in the profile's post list) before chasing it.
- **Follow payoff isn't legible.** After adding people to Following he looked for
  the result "in his feed" and didn't grasp it lives under his user-menu →
  My Circles. Josh wants this treated as part of the larger
  docs/onboarding/direction question, NOT a quick UI patch. Deferred.

**"App vs web" confusion (recurring, expect it from testers):** the web/app
parity work ([[web_app_parity]]) made kwln.city's masthead/layout mirror the RN
app, so testers on kwln.city in a mobile browser (often home-screened) call it
"the app." Tell them apart by wording: web group hero says "SHARE / COPY LINK /
JOIN"; the native group screen says "View Feed / Share / Join" (no Copy Link),
has a back-arrow header, and shares via a popup dialog.
