---
name: feedback_web_feed_composer_match_app
description: "Web feed + post composer must match the mobile app's UX (floating Add-Post FAB over the inline top-of-feed composer); bias to app fidelity over web-only 'richer' divergences."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Directive (2026-08-03): the web feed page and post composer are "completely
different" from the mobile app and must be "identical or as close as possible."
Concrete example Josh gave: the app's **floating Add-Post button (FAB)** instead
of the web's dropdown/inline composer at the top of the feed page.

**Why:** app is the source of truth for UX ([[project_web_app_parity]]). Josh
values screen-for-screen app fidelity over the web having its own (even nicer)
patterns.

**The compose FAB is a SQUARE plum button, NOT a hexagon** (I got this wrong once —
hex is the motif for avatars/type icons, not the action button). Tapping it fans
out the five post types as pills (Event top … Note nearest, each with its hex type
icon); picking one opens the composer preset to that type. The composer header is
an "Add New [Type] ▼" dropdown (all five types incl. Event), NOT a tab row — the
old web tab row overflowed mobile width and hid Event, making Events uncreatable.
Ref: mobile `src/components/nav/ComposeFab.jsx` + `app/compose.js`. Shipped web
version: `frontend/src/components/posts/ComposeFab.jsx` (square + fan-out picker,
`onSelectType`), PostComposer header dropdown.

**How to apply:** when the web diverges from the app on the feed/composer, converge
TOWARD the app — even when an audit flags the web version as "richer" (e.g. the
inline top-of-feed composer, the "All" type-filter button). Replace the inline
composer trigger with an app-style floating Add-Post button that opens the
composer as a modal. Match the app's type-filter solo-on-first-tap behavior
([[feedback_type_filter_solo]]) and view selector. Keep genuinely non-divergent
functional niceties (draft persistence, tags input) but match the app's visual +
interaction model. Desktop-width layout is still a flag-not-guess zone per
frontend/CLAUDE.md — converge the center feed column + composer, confirm before
ripping out the desktop 3-column sidebars.
