---
name: project_web_app_parity
description: "Directive (2026-07-23): bring the web frontend to near-identical UX parity with the mobile app; app is the source of truth."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Josh's directive (2026-07-23): the web frontend (`frontend/`) must reach **near-identical UX parity** with the mobile app (`mobile/`). The **app is the source of truth** — bring the web up to meet it, never change the app to match the web.

Bar is UX parity, not just feature parity: the web should FEEL like the app, screen for screen, tracking the app's 2026-07 redesign ([[project_mobile_redesign_2026_07]]) as closely as the medium allows.

**Three deliberate carve-outs** (platforms genuinely differ, don't force these to match):
- Share (app: native share sheet; web: post-as-Link / copy URL)
- Notifications (app: foreground polling; web: its own delivery)
- Media interactions (native players/gallery vs web `<audio>`/`<video>`/lightbox)

**Judgment-call zone: desktop width.** The app is a phone with a bottom tab bar; a wide desktop browser has to adapt (frontend uses a 12-col sidebar/main/rightsidebar grid + magazine aesthetic per frontend/CLAUDE.md). Flag these spots for Josh rather than guessing.

This directive **overrides** the old frontend/CLAUDE.md rule "there is no generic feed — feeds are always from a Circle." Related: [[feedback_feed_all_posts_merged]], [[feedback_type_filter_solo]], [[project_current_focus]].

## DONE — first parity pass shipped + deployed 2026-07-24 (~02:00 US, overnight)

Built on branch `web-app-parity`, merged to `kowloon-frontend` main (merge bbd8ff5), CI rebuilt `ghcr.io/jzellis/kowloon:latest`, deployed staged to both servers, verified live (grepped served bundle for unique strings — all present). 7 commits, one per area:
1. **Reactions** — one-per-user replace/clear; `ReactButton` now reads `post.myReact`, shows it, clears with `emoji:null`, replaces in one call (mirrors mobile).
2. **Bookmarks edit/move/delete** — new `components/bookmarks/BookmarkActionMenu.jsx` (kebab → Edit/Move/Delete) wired into `UserBookmarksPage` rows + folder headers (owner-only).
3. **Group pending triage** — new `pages/GroupPendingPage.jsx` + `/groups/:id/pending` route + owner Pending button; new `lib/groups.js` (canonical `canJoinGroup`/`joinNeedsApproval`/`rsvpPolicyLabel`) FIXED the `rsvpPolicy==='restricted'` bug that hid Request-to-Join.
4. **Home feed selector** — new `components/posts/FeedViewSelector.jsx`; `feedSlice` gained a general `view` key (all|mine|circleId|groupId, persisted `kowloon:feed:view`, default 'all'); `HomePage` branches fetch across getServerPosts/getUserPosts/getCirclePosts/getGroupPosts.
5. **Discover** — new `components/discover/RecShelf.jsx`+`RecCard.jsx`; `DiscoverPage` fetches `getRecommendations`, renders curated shelves, falls back to popular circles when a server has none.
6. **Profile** — cover image (featuredImage) on edit (`ProfilePage`) + view (`UserPage` 3:1 banner); location display; REMOVED follower/following/posts counts + picsum/mock fallbacks from UserPage.
7. **Search** — bookmark results (type filter + `BookmarkResult`).
Area 8 (auth welcome/onboarding) satisfied without code: RegisterPage already →/discover w/ banner (now curated via #5); web anon `/` is an open public feed, a better web entry than the app's login gate.

**Rollback pin** (pre-deploy image, still on both boxes): `sha256:b7f29817117bea423c7e248762ce4dfef1e3c1e5bf70f4a0da4b5fb601083182`. Instant rollback per server: `ssh jzellis@kwln.social 'cd ~/kowloon[-city] && docker tag <pin> ghcr.io/jzellis/kowloon:latest && docker compose up -d app'`. Or `git revert -m 1 bbd8ff5` in kowloon-frontend + push (rebuilds).

**Deferred (need Josh awake):** Federation/remote surface (area 9 — the big single-origin rework: servers directory, remote profiles, browse remote circles); profile **location EDITOR** on web (display works; web LocationField is composer-shaped); sidebar `DiscoverSection.jsx` still hardcoded (main Discover page done). Couldn't visually QA headless — Josh to eyeball in the morning.

## Follow-on fixes shipped + deployed 2026-07-24 (kowloon-frontend main + server main)
- **SPA content-negotiation (server):** browser visits to `/pages/:slug` and `/circles/:id` returned raw API JSON. Added the `wantsHTML` → `next('router')` guard (mirrors posts/users/groups) so browsers get index.html. `routes/pages/index.js`, `routes/circles/index.js`.
- **Web @domain server search (area 9 slice):** `SearchPage` now detects a bare `@domain` → `feeds.getServer({domain})` → server card → new `pages/ServerPage.jsx` (`/server/:domain`): remote server identity + cached circles/groups/pages + Visit link. `/servers/:domain` is auth-gated + fetches/caches the remote on demand. Remaining federation depth (browse remote LIVE feed cross-origin, join remote circles, remote user nav) still deferred.
- Deploy note: the kowloon-frontend `Trigger Docker build` workflow can fail on a transient GitHub 5xx; recover by manually dispatching `gh workflow run docker.yml --repo jzellis/kowloon --ref main` (bot token has actions:write). Rollback pin from this deploy: `sha256:eff32be4…`.

## DONE — second parity pass (Discover-led full sweep) shipped + deployed 2026-08-03 (kwln.social + kwln.city)

Full side-by-side audit (6 areas) → one prioritized checklist → built in 6 parallel file-disjoint subagents → integrated + built + deployed. Frontend commit `15b648b` (kowloon-frontend main), server commit `082b24bf` (kowloon main). 29 frontend files + 1 server file. Rollback = `git revert 15b648b` (frontend) and/or `082b24bf` (server) + push (CI rebuilds). Verified live: grepped served bundle for unique markers (kowloon:unread, Popular Media, canReply, mediaKind, constrain — all present on both stacks).

What shipped, by area:
- **Discover + server pages** — web `DiscoverMediaTile` (image→post, video→fullscreen modal `<video>`, audio→`AudioPlayer`); `RecShelf` media branch; `RecCard` Server case + circle Save; blurred `discoverBackground` backdrop + onDark panels on `DiscoverPage`; **`ServerPage` rewrite** (16:9 hero, stat pills, Add-to-Circle, Popular Media strip, tabbed Public Posts / Circles / Groups / Pages / remote-Discover); new **`ServersPage`** browse + `/servers` route + Servers nav link (desktop header only — phone bottom bar is full).
- **Composer** (PostComposer + NewPostPage + EditPostPage) — canReply/canReact scope (collapsible Advanced); seed type/audience/reply-react from prefs; client-side upload-size guard; live debounced OG link-preview card + blockquote inject; Event hero image in NewPostPage. Reshare cap (#47) enforced via new `CircleSelector` `constrain="server"` prop (drops the Public option).
- **Settings/profile** — `ProfilePage` preferences now render generically from the shared `@kowloon/client` PREFS manifest (default audience, reply/react scope, home screen, all notification toggles); block/mute on `UserPage`; dead Share button removed.
- **Feed/post** — location line on non-Event posts; in-app SPA linkify for mentions/Kowloon links (anchor-click delegation in `PostBody`); per-emoji `reactCounts` on the detail view; reshare audience-tier gating in `PostToolbar`.
- **Circles/groups** — **circles now default to PRIVATE (self), not @public (privacy bug fixed)**; My/Browse tabs on CirclesPage/GroupsPage; live 350ms member type-ahead; invite-only + circle-scoped groups in GroupForm; group share; CircleCard visibility-label fix (reads `to`, not `visibility`).
- **Notifications/search** — mark-read-on-tap with routing; live unread-badge sync (Header + BottomTabBar listen for a `window` `kowloon:unread` CustomEvent emitted by NotificationsPage); per-type search pagination (Load more / See all).

**Server change:** added the `wantsHTML → next('router')` guard to `routes/servers/index.js` (GET / and /:domain) so `/servers` browser deep-links serve the SPA (was 401 JSON) — same pattern as posts/pages/circles/users.

**Deep-link hard-refresh bug — FIXED 2026-08-03 (server commit `a36a51d1`, deployed both stacks).** All SPA routes that shadow an API path (`/circles`, `/groups`, `/profile`, `/search`, `/notifications`, `/users`, `/posts`, `/pages`, `/admin/*`, and sub-routes like `/circles/:id/posts`, `/users/:id/posts`) returned raw API JSON on a hard refresh / deep-link — only the `/:id` detail routes had per-route guards. Replaced piecemeal guards with ONE centralized guard at the top of `routes/index.js` (before all API subrouters): browser navigations (`Accept: text/html`) under a known SPA path segment defer to the SPA fallback via `next('router')`; ActivityPub / programmatic / `*/*` callers and `?rss` feeds fall through to the API unchanged. `SPA_SEGMENTS` = {circles, groups, users, posts, pages, profile, search, notifications, servers, discover, admin}. Proven with a standalone Express reproduction (19/19) and verified live: browser→SPA, JSON→API, AP actor→activity+json, RSS→text/xml, config.json→json, all good on both stacks. Per-route `wantsHTML` guards left in place (now redundant but harmless). To add a new SPA top-level route that shadows an API path, add its first segment to `SPA_SEGMENTS`.

## DONE — third pass: feed + composer convergence shipped + deployed 2026-08-03 (both stacks)

Josh directive: web feed + composer must match the app ([[feedback_web_feed_composer_match_app]]) — floating Add-Post button over the inline composer; keep the desktop 3-column shell, converge the CENTER column. Re-audited (4 areas) → built in 4 parallel packages → deployed. Frontend commit `3d0ceb4` (kowloon-frontend main), 20 files. Verified live (bundle markers: hex-mask.svg, focusReply, "No replies yet", defaultFeedView, "Mute author", reactCount — all present both stacks).

- **Feed + FAB**: new `ComposeFab` (hexagon-masked floating Add-Post button, sticky in the feed column) replaces the inline top-of-feed composer; `PostComposer` gained a controlled-open/`hideTrigger` mode it drives. Wired on Home (all), circle feed (owner-gated), group feed (member-gated). New shared `TypeFilter` with the app's solo-on-first-tap behavior (no "All" button) unifying the 4 divergent web type-filters. `FeedViewSelector`: refetch on open, search at >0, resolve/enter non-owned circles/groups + Copy. Set-as-default view/filter menu → `prefs.defaultFeedView`/`defaultPostView`. Deleted dead `FeedHeader.jsx`; removed PostList client-side double type-filter.
- **Bugs fixed**: feed cards showed no timestamp (read `post.published`; server sends `publishedAt`/`createdAt` — now `published ?? publishedAt ?? createdAt`); EditPost Event save shifted time (#63 tz drift); full-page composer double-submit (added `dedupeKey`); group multi-type filter returned everything; circle feed composer shown to non-owners.
- **Post cards**: overflow `⋮` menu (report/block/mute/copy link+text/open + owner edit/delete); reaction total next to emoji summary.
- **Post detail**: optimistic reply insert (kills full-page reload + #66 read-lag), history Back, "no replies yet" empty state, `?focusReply` autofocus, `publishedAt` reply-time fallback, removed mock fallback.
- **Composer depth**: EditPost can edit Media attachments + hero image; full rich-text toolbar (lists/strike/code/undo); Event smart date auto-fill in the full-page composers.

**Follow-up flagged:** true multi-type filtering on GROUP feeds needs a server change (`routes/groups/posts.js` does exact `filter.type = query.type`; make it `{$in:[...]}`). Web now sends the first active type for groups — matches mobile's own `useFeed` — so the "2+ types = everything" bug is gone, but selecting 2+ types on a group still shows only the first. Circles already support the `types` array.

**Known limitations / follow-ups (not blockers):**
- Web `CircleSelector` has **no "Only Me" (self/private) audience tier**, so private-audience *post* defaults can't be selected on web (settings + composer). Mobile has it. Deferred.
- Reshare `constrain` not persisted in PostComposer localStorage draft (re-seeds on reopen).
- Verified by build + client-method-signature checks + live-bundle marker grep — NOT runtime-QA'd. Josh to eyeball the interactive flows.
