---
name: feedback-feed-all-posts-merged
description: "Mobile feed shows a single 'Community Posts' view (merged public + server); don't reintroduce separate Public/Server selector rows."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

The mobile feed view selector shows **one** top-level view, "Community Posts" — a
merged public + server firehose for logged-in users. There is intentionally NO
separate "Public" and "Server" choice.

**Why:** The server's `GET /posts` already returns the merged view (`{ $in:
["public","server"] }`) for authed local users when `?to=` is omitted; the
public/server split only ever lived in the mobile client and was clutter.

**How to apply:** `FeedViewSelector` renders a single `serverViews` entry with
`value: "all"`. `useFeed` calls `getServerPosts()` with no `to` for any
non-circle/non-group key. The server still *supports* `?to=public|server` (used
by nothing in the app now) — do not re-expose it as separate selector rows.
Legacy persisted `"public"`/`"server"` viewKeys resolve to the merged view.
Related: [[project-mobile-redesign-2026-07]], [[feedback-type-filter-solo]].
