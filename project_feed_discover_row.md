---
name: project_feed_discover_row
description: "FeedDiscoverRow: shuffled horizontal row of square cards from the Discover pool (GET /recommendations), shown only atop the Community Posts feed. Web live + mobile built 2026-08-06."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

`FeedDiscoverRow` — a single horizontal row of **168px square cards** at the top of
the **Community Posts** feed, pulled from the SAME pool as the Discover page so
admin-curated content shows there too.

- **Data:** `client.feeds.getRecommendations()` → `{ sections:[{ items:[cards] }] }`.
  The endpoint (`routes/recommendations/index.js` `shapeCard`) already shapes each
  item by `refType` (Post/Circle/Group/Bookmark/Page/Server). Flatten all sections,
  dedupe by `refType:id`, **Fisher-Yates shuffle** (Josh wanted types mixed, not
  clustered by section), cap 18.
- **Cards (square):** Post = media/featured image + author overlay, else text
  (type + preview + author); Circle/Group = icon + name + member count + blurb;
  Page/Bookmark/Server = compact. Whole card is the tap target (no per-card
  buttons — trimmed from the Discover `RecCard`).
- **Gating:** shown ONLY when `view/viewKey === 'all' && no active type filter` —
  Community Posts only; hidden on circle/group/My-Posts feeds or when filtering.
- **Web:** `frontend/src/components/discover/FeedDiscoverRow.jsx`, mounted in
  `HomePage` (LIVE both stacks). **Mobile:** `mobile/src/components/discover/
  FeedDiscoverRow.jsx`, mounted as the feed FlatList `ListHeaderComponent` in
  `app/(tabs)/feed.js` (rides the build). Both reuse existing avatar components.

Prototype Josh liked ("it's awesome"). Related: [[project_recommendations_discover]], [[web_app_parity]].
