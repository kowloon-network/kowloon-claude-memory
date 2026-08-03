---
name: project_kowloon_links_in_app
description: How ANY Kowloon item link opens in the mobile app + gets OG kowloon:id meta (issues
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Issues #73/#74 (kowloon-mobile): a link to *any* Kowloon item on *any* server should open in the app, not the browser — server-agnostic (not just kwln.social/kwln.city), because more servers will spin up. Built 2026-07-27 (mobile branch `links-in-app`, pending EAS build; server deployed).

**Server (deployed):**
- `server/methods/seo/meta.js` + `shell.js`: content pages emit `<meta name="kowloon:id">` and `kowloon:type` (post/user/group/page). `renderHeadTags(meta)` extracted for reuse.
- `server/index.js`: SPA fallback `app.get("*")` injects per-item meta into `index.html` for `/posts|users|groups|pages/.+` routes for ALL visitors (not just crawlers) — strips default og/twitter/title, inserts before `</head>`.
- `server/routes/preview/index.js`: added `timeout: 9000` + browser UA (link-preview-js's 3s default was the real cause of preview failures). Added `kowloonRefFromUrl()` — regex-extracts the prefixed id from `/posts/post:...@domain` etc., returns `kowloonId`/`kowloonType`, and fire-and-forget `fetchRemoteServerProfile(kref.domain)` to **passively pull discovered Kowloon servers** into FederatedServer.

**Mobile (branch `links-in-app`):**
- `src/lib/parseKowloonUrl.js`: generalized to posts/circles/groups/users (self-identifying via the prefixed id in the path, so server-agnostic) + pages (only for hosts registered via `registerKowloonHost`, since pages use slugs). Exports `kowloonRouteFromUrl`, `openKowloonLink(href)` (routes in-app or `Linking.openURL`), `registerKowloonHost(domain)`.
- `src/lib/client.js` `ensureClient` calls `registerKowloonHost(account.server)`.
- `HtmlContent.jsx`, `posts/PostCard.jsx`, `posts/PostBody.jsx` use `openKowloonLink`.
- Composer (`app/compose.js`) stores a linked Kowloon item's ID in the Post `target` field via `parseKowloonUrl(linkHref).id`.

**RN URL landmine (fixed 2026-07-27):** `parseKowloonUrl` originally used `new URL(href).pathname`. React Native's built-in URL (NO `react-native-url-polyfill` installed) percent-encodes the `@` in `.pathname` → `/users/@h@d` became `/users/%40h%40d`, so the literal-`@` path regexes never matched and EVERY Kowloon link fell through to the browser (posts/groups/circles too — all carry `@domain`). Fix: parse the raw href string with a regex for scheme/host/path (keeps the literal `@`) + `decodeURIComponent` the path so `%40` forms also match. Don't reintroduce `new URL().pathname` here without adding a polyfill. This surfaced via #77 mentions (`@user@domain` profile links opening in browser).

**Remote pages in-app — DONE 2026-07-28 (server-hydrate, #74):** previously deferred (getPage was local-only). Now: `Page.originDomain` added; `server/methods/pages/hydrateRemotePage.js` fetches the origin's own public `GET /pages/:slug` (JSON via the content-negotiation) and upserts a local Page shadow — PUBLIC pages only, SSRF-guarded (host must already be in `FederatedServer`), 24h freshness, mirrors `hydrateRemoteFile`. `GET /pages/:idOrSlug?domain=<remote>` triggers it. Client `getPage({pageId, domain})`. Mobile: `parseKowloonUrl` threads the host as `?domain=` on the page route → viewer passes it to getPage; and `ensureClient` syncs known Kowloon hosts from `GET /servers` (auth-required; token read lazily from storage) → `registerKowloonHost` each, so remote page URLs get RECOGNIZED (the reason they fell to the browser). Verified live: kwln.social hydrated kwln.city's `welcome-to-kowloon`. TODO(hardening): re-sanitize the hydrated body locally before surfacing hydrated pages on the web.

Key insight: most Kowloon URLs carry the prefixed id IN the path (`post:...@domain`), so they're identifiable without knowing the server. Only pages use bare slugs → need the `kowloon:id` OG meta or a known host. Cross-server page *viewing* in-app is deferred (local-only `getPage`). External OS deep-linking (Android App Links / iOS Universal Links) is deferred — needs native config + a build. See [[feedback_mobile_routing_singular]], [[project_federation_pull_architecture]], [[reference_github_bot_account]].
