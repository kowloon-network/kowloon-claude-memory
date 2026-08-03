---
name: project_mobile_admin_plan
description: "Plan (2026-07-24) for building a server-admin section into the Kowloon mobile app; v1 = dashboard/moderation/users/invites/pages, Discover curation next."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Josh wants a **server-admin section in the mobile app** (`mobile/`) — for many users the app is the ONLY way they experience Kowloon, so admins must be able to run their community from a phone. Design settled 2026-07-24; build not yet started.

**No new plumbing:** the app already ships `client.admin` (AdminClient, ~35 methods) and login/`/auth/me` already surfaces `user.isServerAdmin`. So this is UI + navigation + gating only.

- **Gate:** read `user.isServerAdmin` (from `/auth/me`); show admin entry only to admins. Server enforces `isServerAdmin(actorId)` (membership of the circle in the `adminCircle` setting) on every `/admin/*` call — 403 is the real backstop. No new field needed.
- **Entry point:** BOTH the left drawer AND the user menu (Josh's choice), admins-only. Opens an `app/admin/*` stack (singular routes per [[feedback_mobile_routing_singular]]).
- **Reusable component:** one "admin resource list" (fetch `orderedItems`/`totalItems`, Active/Deleted/All filter, long-press action sheet for deactivate/restore) powers Users/Posts/Groups/Pages/Invites.

**v1 scope:** Dashboard (`serverStats`), Moderation (`getFlagged`/`resolveFlag` + content `delete*`/`restore*`), Users (`getUsers`, `deleteUser`=ban / `restoreUser`, role `addAdmin`/`addMod`), Invites (`createInvite`/`getInvites`/`deleteInvite` + QR share), **Pages** (`getPages`/`createPage`/`updatePage`/`deletePage`/`restorePage`).

**Pages editor:** REUSE the composer's `@10play/tentap-editor` WYSIWYG (`useEditorBridge` + `RichText`, `pmToMarkdown` → `source.content` markdown — server renders to `body` HTML). Page fields: `title`, `slug` (auto), `summary`, `body`, `to` (visibility), `image`, and nav (`type: Page|Folder`, `parentId`, `order`). Works in Expo Go (uses react-native-webview, already used by the composer).

**Discover curation (Phase 2 — designed, not built; NO web UI exists for it either):** shelves = `RecommendationSection` (name/summary/order/active); items = `Recommendation` (a `ref` to an existing Post/Circle/Group/Bookmark/Page + note + order). People are intentionally NOT recommendable — to feature people, curate a server-owned Circle and recommend it. **Add flow = "Feature in Discover" action on any content detail screen** (admins only) → pick shelf + optional note (curate while browsing); the admin Discover screen organizes/prunes. Build the same on web afterward. Rec/section API: `getSections`/`createSection`/`updateSection`/`deleteSection`, `getRecommendations`/`addRecommendation`/`updateRecommendation`/`removeRecommendation`. Discover READ side already shipped ([[project_recommendations_discover]], [[project_web_app_parity]] area 5).

**Gotchas to carry (don't copy web blindly):** Flag schema field is `target` (not `targetId`) and `reason` is an Object; the web invites `active` filter is silently dropped by the client method; verify live server response shapes when wiring. Themes use a separate `client.themes` sub-client, not AdminClient (deferred anyway).

**Deferred (desktop-first):** Themes, full Settings (SMTP/media/profile JSON), Backup/restore, Logs, bulk multi-select. Settings *toggles* (registrationIsOpen, requireEmailVerification, reactEmojis, flagOptions) are an easy Phase-2 add via `updateSetting`.

## v1 BUILT 2026-07-24 — branch `mobile-admin` (pushed, NOT merged; Expo-Go testable)

Files: `src/lib/useIsAdmin.js` (gate), `app/admin/index.js` (dashboard), `app/admin/moderation.js`, `app/admin/users.js`, `app/admin/invites.js`, `app/admin/pages.js` + `app/admin/page/new.js` + `app/admin/page/[id].js`, `src/components/admin/PageForm.jsx`. Entry wired into `UserMenu.jsx` + `LeftDrawer.jsx` (both gated by `useIsAdmin`). Bundles clean via `npx expo export` (my mobile build-check). NOT visually QA'd — Josh to test in Expo Go before an EAS build.

Verified server facts while building: admin gate = `user.isServerAdmin` from `/auth/me`; `client.auth._user.isServerAdmin` set on session restore. `serverStats` → `{counts:{users,posts,groups,circles,pages,replies,reacts,activities,openFlags,activeInvites}}`. `getFlagged` drops the `status` param (hit `/admin/flagged` directly). `/admin/search` is NOT mounted (no adminSearch → used paginated list for Users). Admin route mounts confirmed incl. `/admin/sections` + `/admin/recommendations` (Discover curation API is live for Phase 2). Admin pages route accepts `source` on create+update (pre-save renders markdown→body) but does NOT auto-slug (client slugifies title). Flag `reason` is an Object (render `.label`).

**v1 known scope cuts (call-outs for Josh):** Users has no search (admin/search unmounted) — paginated list only; Pages v1 is flat (no Folder/parentId/order nav UI); Moderation notes on resolve are not prompted (immediate resolve/dismiss). Roles: only add admin/mod (no remove) in v1.

**UPDATE 2026-07-27:** admin v1 **merged to main and included in an EAS dev-client build** (no longer on the `mobile-admin` branch). Live in the app for `isServerAdmin` users.

**NEXT: Discover curation** — still to design the "Feature in Discover" flow with Josh (see this file's Discover section). API is mounted and ready.
