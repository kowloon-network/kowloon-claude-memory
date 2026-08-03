---
name: search-architecture
description: Kowloon search is local-only per-server for 0.1; a third-party network indexer is deferred until after the 0.1 release.
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

Search is **local-only, per-server** for the 0.1 release. Each server searches the content it already holds (locally-owned Posts/Pages/Groups/Bookmarks plus federated content already delivered to its FeedItems cache). No cross-server query fan-out.

**Why:** Server-to-server federated query (fan a query out to the follow/circle graph) was rejected — it leaks privacy-sensitive queries, adds N round-trips of latency, and is a DDoS amplifier. Local-only respects the consent model perfectly and ships with zero new infra (Mongo `$text` indexes). Mastodon ran exactly this for years. Decided 2026-06-20.

**Deferred (post-0.1):** "Option 3" — an independent `kowloon-search` third-party indexer that participating servers push `@public` content to for network-wide discovery. NOT part of 0.1. Don't build it into the `/search` endpoint.

**How it's implemented (server):**
- `routes/search/index.js` — `GET /search?q=&searchIn=posts,users,...`. Per-type consent gate in `typeFilter(typeName, ctx)`:
  - Post / Page / Bookmark → shared `buildVisibilityQuery(ctx)` (same gate as the timeline).
  - User → always discoverable (active, not deleted, not blocked); no `to` filter. Personal-info profile fields gated per viewer via `gateUserProfile()` (added to `methods/sanitize/object.js`). See [[user-profile-visibility-model]].
  - Group → public + same-domain + groups the viewer is a member of.
- Result shape unchanged: ActivityStreams OrderedCollection, each item tagged `_searchType`.
- Bookmark schema got a `$text` index (title/summary/source.content/body/tags). Bookmarks still never federate — search only ever surfaces the owner's own tree. See [[bookmarks-are-personal-only]].

**Mobile search screen** (built 2026-06-20): `mobile/app/search.js` — tabs All / People / Posts / Groups / Bookmarks. "All" shows ALL_PREVIEW (4) of each type with a "See all" jump; type tabs paginate. Debounced 350ms, MIN_QUERY=2. Entry point is a "Search" row in `src/components/UserMenu.jsx` (the avatar dropdown). Reuses PostCard/GroupCard/BookmarkCard/Avatar; users render via a local UserResultRow. Server caps each type at limit*2 (~40) results, so pagination tops out there — fine for 0.1.

**User search uses regex, not $text (fixed 2026-07-02):** The User branch in `routes/search/index.js` breaks out of the `$text` path and does a case-insensitive regex `$or: [{ username: re }, { "profile.name": re }]`. This means "jzell" matches "jzellis". The regex is escape-sanitized before use (`q.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")`).

**Known limitation for non-User types:** Posts/Pages/Groups/Bookmarks still use Mongo `$text` — matching is whole-word. "alic" won't match "alice" in post bodies. Prefix/wildcard search for these types is a deferred enhancement.

**Server (cached-federated) search — issue #3, shipped 2026-07-29:** `/search` now includes a `Server` pass over the `FederatedServer` cache. Partial, case-insensitive regex on domain OR name; leading `@` stripped ("@kwln" and "kwln" both match kwln.social/kwln.city). Excludes `status $nin [suspended, blocked]` (unreachable/slow still show). Server is in the default `ALL_TYPES` set; `searchIn=servers` selects it alone. Results carry `_searchType: "Server"`, `id = domain`, `_score` 12 if `@`-prefixed else 6. Client: `client.search.searchServers({ query })`. This is LOCAL cache lookup only — distinct from `client.feeds.getServer({ domain })` which does a live single-domain fetch (even for uncached servers). Surfaced in both mobile (`app/(tabs)/search.js`, merged with the exact @domain live lookup, deduped by domain) and web (`SearchPage.jsx` Servers section on the All tab). This is the discovery building block; the fuller remote-server **discovery page** (#2) and **onboarding** (#1) are deferred pending Josh's design — do not start them without direction.
