---
name: mobile-share-intake-todo
description: Android share-into-Kowloon is BUILT & working (share URL/text/image from any app -> prefilled composer). iOS Share Extension still TODO. Plus the EAS-build gotchas learned building it.
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Inbound share-into-Kowloon: OS share sheet -> Kowloon -> prefilled composer.
**Android: DONE and verified 2026-07-17** (Chrome/Firefox/Feedly URLs -> Link,
selected text -> Note, gallery image -> Media). **iOS: still TODO** (Share
Extension + App Group `group.social.kwln.share` + TestFlight).

**How it's wired (mobile):**
- `expo-share-intent` **pinned to 6.x** (`^6`) — v8 needs SDK 57, we're on SDK 55
  (per its compat table). `npx expo install` wrongly grabs v8; pin manually.
- `app/_layout.js` wraps the app in `<ShareIntentProvider>`; `ShareIntentRouter`
  (mounted globally) consumes `useShareIntentContext()` and routes to `/compose`.
  It's inert in Expo Go (native module is `requireOptionalNativeModule` -> null).
- Config plugin in `app.json`: iOS activation rules + `iosAppGroupIdentifier`,
  Android intent filters (text/image/video). Composer intake:
  URL -> `?type=Link&href=`; text/files -> `pendingShare` + `?fromShare=1`.

**Three real bugs, three real fixes (don't regress):**
1. **navigate-before-mount**: routing on cold start threw "Attempted to navigate
   before mounting the Root Layout". Fix: gate on `useRootNavigationState().key`
   AND `setTimeout(..., 0)` the `router.push`, plus a handled-guard ref.
   *(Superseded 2026-08-05 by the reliability rewrite below — the router now
   gates on nav key + account hydration and retries, no setTimeout defer.)*
2. **text/plain-only native gate**: expo-share-intent's Android handler drops any
   `ACTION_SEND` whose MIME isn't `text/plain` (Firefox/Feedly send `text/html`
   or null type). Fix = `patch-package` patch (`patches/expo-share-intent+6.1.1
   .patch`, applied via `postinstall`) that accepts any ACTION_SEND carrying text
   in `EXTRA_TEXT` **or** `ClipData`.

**EAS build gotchas (this repo) — see [[project-production-servers]] cousin:**
- `@kowloon/client` is a `file:../client` sibling; EAS only uploads the mobile
  repo, so bundling died with `ENOENT .../client`. Fix: `eas-build-pre-install`
  npm script clones the PUBLIC `kowloon-client` repo into `../client` on the
  build server. **Push client changes before any EAS build** (it clones `main`).
- Debug native share issues with a **dev-client** build (`--profile development`)
  connected to Metro — `debug:true` on the provider streams the share payload to
  the Metro logs; JS fixes hot-reload (no rebuild). Standalone/preview builds
  hide all logs. Native (.kt/patch) changes still need a rebuild.
- Josh upgraded to **EAS priority builds** (free tier queued ~1h+); builds now
  start in seconds. eas.json `preview`/`development` output APKs.

**Share flow v2 — #81 (route-independence) + #82 (destination chooser), 2026-07-31 (in the links-in-app build):**
- **#81 fix:** the old `ShareIntentRouter` boolean "handled" flag could stick `true` and swallow the next share (symptom: sharing worked only from the Feed screen, did nothing from a post). Replaced with a **content-keyed dedupe** (`shareKey(si)` = url/text/files), `router.navigate` (not push), and an **AppState "active" re-check** — a shared URL now routes from ANY screen. Don't reintroduce a boolean handled-flag.
- **#82:** a shared **URL** now opens a chooser screen `app/share.js` (`/share?url=`) — **Create a Link post** (`/compose?type=Link&href=`) or **Save as a bookmark** (routes to the existing `BookmarkComposer` with `initialValues.href` prefilled + locked). Text/file shares are unchanged (straight to the Note/Media composer). `targetFor()` in ShareIntentRouter sends URLs to `/share`, everything else to `/compose`.

**Share reliability rewrite — 2026-08-05 (links-in-app, unbuilt at write time):**
Tester hit flaky shares: sometimes worked, sometimes dropped, sometimes the Link
page replayed on the next few app opens. Root cause = a load-order race. Fixes in
`ShareIntentRouter.jsx` + new `src/lib/shareDedupe.js`:
- **Drop fix:** the old handler `resetShareIntent()`'d even when `router.navigate`
  threw (nav not ready), consuming a share it never routed. Now it consumes/resets
  ONLY after a confirmed-successful navigate, and retries (bounded) otherwise.
- **Auth race:** delivery now waits for `selectAccountsStatus === 'ready'|'error'`
  (accounts hydrated) as well as the nav key, so a share never routes into a
  screen that bounces to /welcome.
- **Replay fix:** the in-memory `lastKey` ref reset every cold start, so Android's
  recents-replay of the launch intent re-fired the share. Persist the last
  delivered content key to AsyncStorage (`shareDedupe.js`); a matching inbound
  share is a replay and is dropped. Marker is cleared on a clean launch (no share,
  1.2s debounce to dodge the cold-start no-share blip) so re-sharing works later.
- Keep `<ShareIntentProvider options={{ resetOnBackground: false }}>` — it means an
  UNdelivered share survives a brief background; the router resets explicitly once
  delivered.

**Regression + fix 2026-08-06 (unbuilt at write time):** that rewrite ALSO dropped
the old `setTimeout(0)` before `router.navigate`, which broke sharing ENTIRELY
(opened the app, did nothing). A synchronous navigate on cold start silently
no-ops; the "no throw = success" logic then consumed the share. Fix: DEFER the
navigate (setTimeout), consume/reset ONLY on a real navigate, non-blocking marker
load, in-flight guard. Same cold-start-nav trap as the push-tap fix — see
[[feedback_mobile_coldstart_nav]].

Related: [[project-mobile-app-scaffold]], [[project-mobile-strategy]], [[audio-player]].
