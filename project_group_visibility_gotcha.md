---
name: project_group_visibility_gotcha
description: "A Group's members-circle id lives at circles.members, NOT a top-level members field — querying `members` silently matches nothing (group posts 403'd for members). Fixed 2026-08-06."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

**A Group has NO top-level `members` field.** Its five system circles live under
`group.circles.{admins,moderators,members,blocked,pending}` — each a circle id
string (see `schema/Group.js`). So `Group.find({ members: <circleId> })` matches
NOTHING and returns `[]` silently.

This bit the visibility system: `methods/visibility/context.js` `getViewerContext`
built `ctx.groupIds` via `Group.find({ members: { $in: memberCircleIds } })`, which
always returned empty → the group-post check in `visibility/helpers.js`
(`ctx.groupIds.has(to)`) was always false → **any group-addressed post fetched by
id (`GET /posts/:id`, e.g. a notification tap or shared link) returned 403 "Access
denied" even to actual members.** It only surfaced on single-post views; the group
*feed* endpoint (`/groups/:id/posts`) has its own membership check. Fix: query
`Group.find({ "circles.members": { $in: memberCircleIds } })`.

**Rule:** to resolve a viewer's group memberships, find the circles they're in
(`Circle.find({ "members.id": viewerId })`) then match `Group` by
**`circles.members`** (or admins/mods as needed) — never a bare `members` field.

Related: [[project_following_circle_gotcha]], [[project_codebase_gotchas]], [[project_circle_mosaic_icon]].
