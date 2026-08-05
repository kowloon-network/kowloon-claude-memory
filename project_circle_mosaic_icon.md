---
name: circle-mosaic-icon
description: Auto-generated hexagon mosaic Circle icon composited server-side from member avatars when no creator icon is supplied.
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Circles without a creator-supplied icon get an auto-composited **hexagon mosaic** built from the avatars of their first six members. Shipped + deployed both stacks 2026-07-29; backfilled (kwln.social 6, kwln.city 25 circles).

**Design (settled with Josh, don't re-litigate):** flat-top hexagon, **seamless** (no seam lines — he explicitly rejected them). Layouts by member count: 1=whole, 2=top/bottom halves, 3=top-left/top-right/bottom, 4=quadrants, 5=six wedges with the top-left slot solid **black**, 6=six wedges. Each segment is filled from the **top-centre** of its own avatar (sharp `position:"top"`) so a low segment still shows a head, not a body. Members with no avatar get a neutral grey panel (`#C9C9CF`). 7+ members: only the first six are used.

**Code:**
- `methods/circles/generateCircleIcon.js` — pure sharp compositing, avatar Buffers -> 512x512 RGBA PNG. No DB/network. (Geometry constants + `layout(n)` here.)
- `methods/circles/regenerateCircleIcon.js` — orchestration: resolve each member avatar to bytes (local `/files/` URLs read straight from storage by File.storageKey; remote/other via SSRF-guarded size-capped fetch; static placeholder svgs -> null), composite, store as a File (two-save pattern, `to:@public`, **NO `parentObject`** — see Privacy fix below), `Circle.updateOne` sets `icon` + `iconGenerated:true`, best-effort delete the previous generated file. Self-guards; never throws to caller.
- `scripts/backfill-circle-icons.js` — one-time backfill (`--write`, `--limit`). `loadSettings(Settings)` needs the model arg.

**iconGenerated flag** (new on Circle schema) distinguishes auto vs creator-supplied. Regeneration is eligible when `iconGenerated===true` OR icon is unset/placeholder OR **the icon merely matches a current member's avatar URL** (URL-encoding normalized) — the last covers the legacy "first member's avatar" default the mobile/web clients still SEND in the createCircle payload (the server stores whatever the client sends). It **never overwrites a genuine custom icon**. The Update handler clears `iconGenerated` to false whenever a user sets their own Circle icon.

**Triggers:** ActivityParser `Create` (for `type:"Circle"`), `Add`, and `Remove` handlers call `regenerateCircleIcon(id).catch(()=>{})` fire-and-forget. Create is essential — making a circle WITH members goes through Create, not Add, so without it a new circle keeps the client's first-member-avatar. Add/Remove fire only for `ownerType==="User"` when a member was actually added/removed. Because it's fire-and-forget, a newly created circle shows the first-member avatar for ~1s until the mosaic lands (refresh to see it). Scope guard inside: local `type:"Circle"` circles only (System circles like Blocked/Muted are type System -> skipped). **The Following circle IS in scope** — it's `type:"Circle"` per [[following-circle-gotcha]], so it gets a mosaic of the people you follow (private to owner).

**Privacy (REVISED 2026-08-05 — the earlier inheritance approach was a bug):** the icon File used to set `parentObject:circle.id`, so `serve.js` (`effectiveTo = parentTo ?? file.to`) inherited the **circle's** visibility. That broke EVERY user's Following mosaic: Following is self-addressed (`to:@<owner>`), so its icon served **401 even to the owner** — clients load the icon as a plain `<img>`/`<Image>` URL with no auth/HMAC signature, so a restricted file just fails. "Privacy-correct" in theory, unloadable in practice. **Fix:** drop `parentObject` — the icon stands as the `@public` file it's created as. No real leak: the icon is a composite of already-public avatars, and its `file:<mongoid>@domain` URL is opaque + only reachable via the circle object itself (as private as the circle's existence). Backfilled all existing generated icons on both DBs (`db.circles.find({iconGenerated:true, icon:/\/files\/file:/})` → set the icon File `to:@public`, `$unset parentObject`). Custom-uploaded icons for private circles could hit the same 401 — not addressed (out of scope; only mosaics were backfilled).

**Staleness (Josh's call):** regenerate on membership change only, NOT when a member later swaps their own avatar. Mild staleness accepted.

Cost: CPU-only (sharp/libvips, no GPU), paid once per membership change, dominated by fetching not-yet-local avatars. Prototype + generator live in the session scratchpad `hex-proto/`.
