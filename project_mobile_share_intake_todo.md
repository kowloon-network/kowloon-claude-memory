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

Related: [[project-mobile-app-scaffold]], [[project-mobile-strategy]], [[audio-player]].
