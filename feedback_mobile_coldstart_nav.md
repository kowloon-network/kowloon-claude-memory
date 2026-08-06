---
name: feedback_mobile_coldstart_nav
description: "Mobile: never router.navigate synchronously on a cold-start deep-link (share intent, push tap) — it silently no-ops/throws before the navigator mounts. Gate on useRootNavigationState().key + defer."
metadata: 
  node_type: memory
  metadata_type: feedback
  type: feedback
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

**Any code that deep-links on app launch (OS share intent, push-notification tap,
cold-start URL) must wait for the navigator before `router.navigate/push`.** A
synchronous navigate the instant the app cold-starts fires BEFORE the expo-router
`<Stack>` has mounted: it either throws "Attempted to navigate before mounting the
Root Layout" OR — worse — **silently no-ops** (no throw). If your code treats
"didn't throw" as success and then consumes the intent, the deep-link is lost and
the app just opens on the default screen.

Two real regressions from this, both fixed 2026-08-06:
- **Share intent** (`ShareIntentRouter.jsx`): a rewrite dropped the `setTimeout(0)`
  before navigate; the sync navigate no-op'd, was counted as success, and reset the
  share → sharing opened the app and did nothing. See [[project_mobile_share_intake_todo]].
- **Push tap** (`PushProvider.jsx`): it wraps the `<Stack>`, so a cold-start tap
  navigated before mount → opened the default screen instead of the post.

**The pattern:** read `const navReady = !!useRootNavigationState()?.key`. Navigate
only when ready; otherwise stash the target route in a ref and flush it in an
effect keyed on `navReady`. Wrap the actual navigate in `setTimeout(…, 0)` so it
lands after the current commit. Consume/reset the intent ONLY after a genuine
navigate (never on a no-op). Note both `ShareIntentRouter` and `PushProvider` sit
ABOVE/beside the Stack in `app/_layout.js`, which is exactly why they race it.
