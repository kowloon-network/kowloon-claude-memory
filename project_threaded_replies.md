---
name: project_threaded_replies
description: "Two-level (Facebook-style) threaded replies — Reply model schema/handler, client inReplyTo, and the web/mobile tree UIs. Shipped 2026-08-03."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Replies are threaded **two levels deep, Facebook-style** (a reply to a post, and
replies to those — no deeper). Built on the existing separate **`Reply` model**
(NOT a Post subtype — see server/CLAUDE.md "DO NOT CHANGE"; the client's `reply()`
sends `{type:'Reply', objectType:'Reply', to}`).

**Data model (`schema/Reply.js` + `ActivityParser/handlers/Reply/index.js`):**
- `target` = the ROOT post (same for EVERY reply in a thread) — so
  `GET /posts/:id/replies` returns the whole thread in ONE query (`{target: postId}`)
  and federation routes to the post's host.
- `parent` = the IMMEDIATE parent: the post id for a first-level reply, a reply id
  for a second-level reply. Depth capped at 2: replying to a second-level reply
  **flattens** onto its first-level ancestor (handler reads `parentReply.parent`).
- Added `replyCount` to the Reply schema — second-level children count on a
  first-level reply. The root Post's `replyCount` is the TOTAL (all levels).
- Notification goes to the author of the thing actually replied to (immediate
  parent), routed (objectId) to the root post.
- Top-level replies behave exactly as before (backward-compatible); pre-existing
  replies already had `parent === target === postId` from the old pre-save default.

**Client:** `reply({ postId, inReplyTo, content })` — `inReplyTo` (a first-level
reply id) is optional; `to = inReplyTo || postId`; the server derives root + caps depth.

**Web UI:** shared `frontend/src/lib/replyTree.js` `buildReplyTree(replies, postId)`
builds first-level (`parent === postId`) + nested children; used by BOTH
`pages/PostPage.jsx` AND `components/posts/ReplyModal.jsx` (the feed pop-up — it's
a SEPARATE surface, easy to forget). First-level replies get a Reply button;
second-level get `showReply={false}` (no button → depth cap). Orphans (parent
deleted) surface at top level.

**Mobile UI:** same 2-level tree in `app/post/[id]/index.js` + `Reply.jsx` +
`ReplyComposer.jsx` (`inReplyTo` prop).

**ReplyModal mobile gotcha (fixed):** a `fixed top-4 bottom-4` panel anchors to the
mobile browser's LAYOUT viewport (taller than visible when the URL bar shows),
pushing the sticky composer below the fold — cap with `max-h-[calc(100dvh-2rem)]`,
give the scroll list `min-h-0`, `shrink-0` the header/composer.

Related: [[project_reply_federation_lag_possibly_todo]], [[project_web_app_parity]].
