---
name: project_feed_selector_pins
description: "Pin circles/groups to the top of the feed selector — stored in User.prefs (Kowloon IDs), sortByPins shared util, honored in every own-circle list. Built 2026-08-05."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Users can pin circles/groups to the top of the feed-selector sections (and the
pin order is honored everywhere their own circles are listed).

**Storage: `User.prefs`, NOT the Circle/Group docs.** Two ordered arrays of
**Kowloon IDs** (`circle:…@domain` / `group:…@domain`, first = topmost):
`prefs.pinnedCircles`, `prefs.pinnedGroups`. Why prefs: groups are shared, so a
`pinned` field on the Group doc would be global — pinning is a per-viewer pref;
circles go there too for one mechanism; an ordered array gives pin order for
free (drag-reorder is a v2). Added to the `prefs` subschema in `server/schema/
User.js` (real subschema — needs a schema change for new fields). The Update
handler dot-paths `prefs.*`, so `updateProfile({prefs:{pinnedCircles:[...]}})`
replaces the array cleanly.

**Default Following pin:** seeded into `pinnedCircles` at registration (User.js
pre-save) AND backfilled for all existing users (`db.users … $set pinnedCircles
= [following].concat(cur)` on both DBs). Single source of truth; user can unpin.

**Shared code (`@kowloon/client/src/prefs/pins.js`):** `pin/unpin/togglePin/
isPinned/sortByPins(items, pinned)`. `activities.setPins({circles?,groups?})`
persists the full arrays. Both exported from the client's main index.

**Surfaces honoring pins (all via sortByPins):**
- Web: `FeedViewSelector` (pin toggle per row, optimistic via `patchUser`),
  `AddToCircleButton`, composer audience (`PostComposer`→`CircleSelector`),
  `CirclesPage` "My Circles".
- Mobile: `FeedViewSelector` (pin toggle), and the central `orderUserCircles`
  (`src/lib/orderCircles.js`) now takes `pinnedCircles` and sortByPins-es —
  covering `AudienceSelector`; plus `ProfileActions` (Add-to-circle) and the
  Circles tab. Mobile reads pins from `client.auth.getUser().prefs` (loaded at
  init; same-session cross-surface live-sync deferred).
- Browse/discovery lists (other people's, public circles) are NOT pin-sorted.

Related: [[project_mobile_redesign_2026_07]], [[web_app_parity]], [[following_circle_gotcha]], [[project_preferences_screen]].
