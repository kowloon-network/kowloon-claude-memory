---
name: feedback_web_mobile_viewport
description: "Web frontend on mobile: use svh (not h-screen/100vh) for the app shell so the bottom nav labels aren't clipped; safe-area inset only in standalone PWA."
metadata: 
  node_type: memory
  metadata_type: feedback
  type: feedback
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Web frontend, mobile browser gotchas (both hit + fixed 2026-08-05):

**Use `h-[100svh]`, not `h-screen` (100vh), for the app shell** (`Layout.jsx` /
`PublicLayout.jsx`). The feed scrolls INTERNALLY (main has `overflow-y-auto`; the
document doesn't scroll), so the mobile browser's URL bar never hides → the
visible viewport stays the *small* viewport. `100vh` is the *large* viewport, so
the bottom of a fixed bottom-nav (its tab labels under the icons) sat below the
fold and looked like "the icons have no labels." `svh` fits the actually-visible
area. **Why:** internal-scroll layouts don't trigger URL-bar auto-hide, so vh
over-measures.

**`env(safe-area-inset-bottom)` only in standalone PWA.** Some Androids report a
non-zero bottom inset even in a normal browser tab (where the viewport already
sits above the gesture bar), leaving dead white space under the nav. Scope it:
`.pb-safe-standalone { padding-bottom: 0 } @media (display-mode: standalone){ …:
padding-bottom: env(safe-area-inset-bottom) }` (in `index.css`), class on the
`BottomTabBar` nav.

**How to apply:** these apply to any full-height mobile-web chrome pinned to the
viewport edges (bottom nav, sticky headers). Related: [[web_app_parity]].
