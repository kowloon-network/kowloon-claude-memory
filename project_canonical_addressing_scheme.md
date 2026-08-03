---
name: project_canonical_addressing_scheme
description: "The strict canonical `to`/canReply/canReact addressing scheme for source objects, enforced on write and backfilled 2026-07-23."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Josh set a STRICT canonical addressing scheme (2026-07-23) for the `to` (and
`canReply`/`canReact`) fields on **source objects** — Post, Circle, Group, User,
File:

- `@public` — public
- `@<domain>` — server-only (e.g. `@kwln.social`)
- `circle:<id>@<domain>` — a circle
- `group:<id>@<domain>` — a group (NO leading `@` — Josh confirmed)
- `@<owner>@<domain>` — private (a single user)

Enforced on write via `methods/parse/canonicalTo.js` (`canonicalTo` + `canonicalAudience`),
wired into the Create handler, Update handler, and `routes/files/upload.js`. It's
CONSERVATIVE: coerces loose synonyms (`""`, `"public"`, `"server"`, `"true"`) but
preserves anything already id-shaped (`@…`, `circle:`, `group:`) so a private/
circle/group audience is never widened to public. Blank `canReply`/`canReact`
inherit the object's `to`; a blank Circle `to` defaults to the owner.

Existing rows were backfilled on BOTH servers 2026-07-23 (social: 9 posts, 2
files, 4 circles; city: 46 posts, 2 files, 2 circles). Re-audit showed 0 loose
values remaining.

**IMPORTANT exception:** `FeedItems.to` is deliberately a COARSE enum
(`public`/`server`/`audience`) and must NOT carry circle/group ids — it's the
denormalized feed, and leaking circle/group ids there is a privacy bug. Never run
`canonicalTo` on FeedItems.to; `FeedFanOut` is the real grant. See [[project_codebase_gotchas]].

The federation READ layer stays lenient (`isPublicVisibility` accepts `public`/
`@public`/empty) — remote peers may still send loose values we can't control.
This whole scheme grew out of #71 (a bare `"public"` file `to` broke cross-server
media caching). Related: [[project_cross_server_media_and_launch]], [[feedback_actor_canonical]].

Done 2026-07-23: renamed the malformed `@Michelle Ellis@kwln.social` (space in
handle) in place to `@michelleellis@kwln.social` (username `michelleellis`,
display name kept "Michelle Ellis") — user + 5 circles (actorId/to) + 2 member
subdocs; no content to move. She must now log in with `michelleellis`, not the
old spaced handle (password unchanged). Registration validation (`/^[a-z0-9_]{2,32}$/`)
already blocks new spaced usernames, so this shouldn't recur.
