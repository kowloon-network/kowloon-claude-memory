---
name: project_preferences_screen
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Issue #78: a preferences control panel exposing everything in `User.prefs`. Built + deployed 2026-07-27 (server timezone field live on both servers; mobile screen on branch `links-in-app`).

**Architecture decision (Josh weighed the Settings-style `{label,value}` shape and we rejected it):** preferences stay as FLAT values on `user.prefs[key]`. The UI metadata (label, control type, options, group, order) lives in ONE shared manifest, NOT wrapped into each user doc — because that metadata is identical for every user (would bloat every doc + leak into federation), and wrapping would force every consumer from `prefs.x` to `prefs.x.value`. Settings gets away with `{ui,value}` only because Settings is a singleton collection.

- **Manifest**: `client/src/prefs/manifest.js` (re-exported from `client/src/index.js`) → `PREFS`, `PREF_GROUPS`, `getPrefValue(prefs, entry)`. Pure data, no React. Each entry: `{ key, group, type, label, hint?, options?, default }`. `type` ∈ toggle | select | multiselect | audience | timezone. Nested keys use dot-paths (`notifications.mention`). Shared by web + mobile (mobile reads it via the `@kowloon/client` symlink → Fast Refresh, no publish; web bundles it via `file:../client` on next frontend build).
- **Mobile screen**: `app/settings/preferences.js` renders generically per `type` (reuses `AudienceSelector` for audience; `src/lib/timezones.js` for the tz picker — device default + searchable IANA list w/ Hermes-safe fallback). Reached from the settings hub AND a link in the profile editor. Reading typography stays its own screen (linked, not inlined).

**Write gotcha (Update handler, ActivityParser/handlers/Update/index.js ~272): it dot-paths each TOP-LEVEL prefs key** (`patch['prefs.'+k]=v`), so scalars merge but a nested object (`notifications`, `typography`) is REPLACED WHOLESALE. So the notifications section must send the COMPLETE notifications object on every toggle (like typography does). Write via `client.activities.updateProfile({ updates: { prefs: {...} } })`.

**Live vs dead prefs** (audit): LIVE = defaultPostType (web composer), defaultPostView + defaultFeedView (mobile feed bar reads+writes), defaultCircleView (web add-to-circle), notifications.* (server gates), typography.* (mobile reading). DEAD/settable-but-unconsumed = defaultTo, defaultcanReply, defaultcanReact, defaultEditorType (internal), lang (no i18n), theme (mobile has no dark mode). `notifications.follow` is vestigial (Kowloon never notifies on circle-add — [[feedback_no_follow_notifications]]) and is OMITTED from the manifest. Plan: wire the dead ones' consumption now that users can set them. Related: [[project_new_post_notifications]], [[project_mentions]], [[feedback_mobile_routing_singular]].
