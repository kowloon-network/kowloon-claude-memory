---
name: group-feed-member-gating-todo
description: "DEFERRED — main feed timeline should only ever show a group you belong to; Discover should link to the group page, not its feed. Toolbar Join button already removed."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Product decision (2026-07-18, Josh): the main Feed page should **never** show a
Group's feed unless the viewer is a member of that group.

**Done:** removed the toolbar "Join" button from `FeedViewAction` (the group
branch) — it only existed to let a non-member join a group whose feed they were
previewing.

**Still TODO (Josh said not yet):**
- **Discover** (`src/components/discover/RecCard.jsx`) currently routes recommended
  items to `/feed?view=<id>`. For groups it should push the group **page**
  (`/group/<id>`), not the feed.
- **Group detail "View Feed"** (`app/group/[id]/index.js`, ~line 239) is shown to
  everyone and pushes `/feed?view=group:...`. Gate it to members (owner/member),
  or otherwise ensure a non-member can't land on a group feed in the main timeline.
- Once those paths are closed, a group `viewKey` in the main feed always implies
  membership — the removed Join button stays unnecessary.

Note: the feed selector's "Your groups" already lists only joined groups, so the
selector was never a leak. Related: [[project-recommendations-discover]],
[[project-mobile-groups-moderation-todo]].
