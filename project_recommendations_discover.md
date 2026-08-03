---
name: recommendations-discover-architecture
description: "The curated Discover system: RecommendationSection + Recommendation content types, viewer-aware read, server-owned circles for people. Built + deployed 2026-07-16."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Kowloon's **Discover** screen is driven by two server-owned content types (built
2026-07-16, live on kwln.social).

**Data model** (`server/schema/`):
- `RecommendationSection` — a named shelf ("Posts We Love"), with `order`,
  `active`, `to` (section visibility). Server-owned + signed like a Page.
- `Recommendation` — a *reference* (not a copy) to a public/server-visible
  object, with `ref`, `refType` (derived from the ID prefix), `section`, `note`
  (editorial blurb), `order`, `active`, `visibility` (inherited from the target
  at add time). Both default `actorId` to the server actor.
- **Users are NOT recommendable.** Curated *people* are a server-owned Circle
  (allowed — `Circle.actorId` is any string, and `signAs` already signs as the
  server actor), which is then recommended as a `circle:` ref. One mechanism.

**Read** — `GET /recommendations` (`routes/recommendations/index.js`), viewer-
aware: authenticated LOCAL users get public + server-tier items, everyone else
(logged-out, remote, other servers) gets public only — same tier gate as
`/circles` and `/posts`. References are resolved live and **dropped if the
target was deleted or its visibility narrowed** after being curated. File-ID
images are resolved to served URLs per item. Returns `{ sections: [{ name,
summary, items:[card] }] }` where each card is shaped by `refType`.

**Admin CRUD** — `/admin/sections` + `/admin/recommendations` (add-time
validation: curatable type + public/server visibility). No admin web UI yet —
seed via `scripts/seed-recommendations.js` (populates from top public content).

**Client** — `feeds.getRecommendations()`; `AdminClient` has section + rec CRUD.
**Mobile** — `app/discover.js` renders sections as horizontal shelves via
`RecShelf`/`RecCard` (image-forward Post cards, Circle cards with View/Save,
Group cards with View/Posts, link cards). Reached from the feed selector's
"Discover..." footer.

**Sections are the organizing unit** — the flat "one shelf per type" idea was
superseded by named sections from the start (cheap, and the read groups by
section). See [[feedback-feed-all-posts-merged]] neighbours for other Discover
decisions.

**Schema gotcha learned here:** a `required: true` field whose value is set in a
`pre('save')` hook FAILS validation — Mongoose validates *before* pre-save. Set
it with `default: () => ...` (runs at construction) instead, like `originDomain`.

**Typed-rows redesign — Stage 1 built + deployed 2026-07-30 (kwln.social + kwln.city):**
Josh reversed the "named sections" idea to **one row per content type**. `RecommendationSection` gained `contentType` (media|posts|circles|groups|servers, one row each), `source` (curated|heuristic|hybrid, **default hybrid**), `targetCount`. `Recommendation.refType` now recognizes a bare `@domain` as a **Server** (resolved from the FederatedServer cache in the read route; servers are public, no `to`-gate). Post cards expose `mediaImage` (featured, else first attachment) for the media row. `scripts/seed-discover-sections.js` seeds the 5 canonical rows idempotently (run on both stacks). Mobile: `RecCard` gained `MediaCard` (thumbnail grid → post, play badge for video) + `ServerCard` (→ /server/[domain]); `RecShelf` renders media layout when `contentType==="media"`; Discover screen has a blurred server-hero background ([[discover-background]] via `discoverBackground` setting) + translucent panels + white text. Starter content seeded on kwln.social so rows render.

**Locked design decisions:** (1) one type per row; (2) one row per type — "Add to Discover" is unambiguous, no prompt; (3) daily-seeded randomness; (4) curated picks bypass the completeness bias.

**Stage 3 built + deployed 2026-07-30 (both stacks):** `methods/discover/heuristic.js` — posts/media scored by time-decayed `reactCount·1 + replyCount·2` (DECAY_K=12h, GRAVITY=1.4); circles/groups by memberCount; servers by localFollowerCount·3 + userCount; ×1.5 soft completeness boost (real icon+description, or featured image / has-attachment — never a filter); daily-seeded (UTC) mulberry32 shuffle of the top-40 pool; viewer-tier aware (to:@public, +@domain for local); pools cached 5 min per (type, tier, day). `GET /recommendations` backfills hybrid/heuristic rows to `targetCount` after curated picks (curated first, bypass boost, excluded from the pool). Verified pure-heuristic on both stacks (demo picks removed). Rows self-populate with no curation.

**Known edge:** the circles row can surface a public circle literally named "Following" (seed data on kwln.city where such circles are @public). A real system Following circle is self-addressed/private and won't appear (the to:@public filter excludes it). If unwanted, exclude circles that are any user's `circles.following` from the circles pool.

**Remaining — Stage 2 (deferred by Josh; he's the sole admin on both live servers and doesn't want to touch curation yet):** in-context "Add to Discover" action (admin-only via useIsAdmin) + a mobile admin Discover screen (reorder items/rows, remove, set source/target/active) + client AdminClient wrappers. Mobile Stage-1 cards (MediaCard/ServerCard/layout) are in the dev app on reload but need the next EAS build to reach testers.

**Also built 2026-07-31 (all in links-in-app):**
- **Discover background** — `discoverBackground` setting = blurred+darkened server hero (sharp, blur 6 + light gradient, Klein-blue fallback), generated by `methods/discover/generateDiscoverBackground.js`; served in `/recommendations` as `background` AND in `getServerInfo().settings`. Local Discover screen renders it behind translucent panels + white text.
- **Remote-server Discover** — the mobile server page (`app/server/[domain].js`) has a **Discover tab** that fetches that server's public `/recommendations` directly and renders its shelves over ITS blurred hero (onDark). Default tab is **Public Posts** (relabeled Posts firehose); a **"Popular Media"** strip (all media, 4-across, gapless, "Discover More" -> Discover tab) sits between the masthead and the tab bar.
- **Media kinds/players** — `/recommendations` resolves each media item's first-attachment kind via a File lookup and exposes `mediaKind` (image/video/audio) + `mediaUrl` + `mediaName`. `DiscoverMediaTile` renders: image->post, video->fullscreen `expo-video`, audio->the global [[audio-player]]. Post cards also expose `preview` (textPreview) so Note cards show body text.
- **UI**: selected tab labels are **bold** (not just color) across the app's color-only selectors + shared SegmentedControl; server tab bar scroll-selected-into-view.

Related: [[feedback-actor-canonical]], [[signing]], [[search-architecture]] (server cache), [[audio-player]].
