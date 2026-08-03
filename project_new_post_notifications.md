---
name: project_new_post_notifications
description: "Schedulable/cooldown notifications + the \"new posts in your feeds/groups\" nudge (issue"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Issue #75: notify users when a NEW BATCH of posts arrives in their unread feeds — not one-per-post. Built + deployed 2026-07-27. Josh's rules (verbatim intent): "Your feeds have new posts" nudge, at most once per 12h and ONLY if the previous one has been read; separate per-group nudges ("There are new posts in 'Watford FC'"), same rules; feed = everything EXCEPT groups (forget Circles here). Default **ON**.

**General mechanism — `server/methods/notifications/create.js` gained a `cooldownMs` param.** When set, the notification fires only if the latest existing notification with the same `groupKey` is BOTH read AND older than `cooldownMs` (schedulable "fire-only-if-latest-read-and-stale"). Without `cooldownMs` it keeps the legacy 24h dedup. This is the reusable "conditional/schedulable notification" primitive Josh asked for.

**`server/methods/notifications/notifyFeedActivity.js` (NEW):**
- Feed nudge — `groupKey: "feed"`, "Your feeds have new posts", 12h cooldown. Recipients = followers of the author: `Circle.find({"members.id": authorId})` for public/server posts; circle members for circle-addressed.
- Group nudge — `groupKey: "group_posts:<groupId>"`, "There are new posts in '<name>'", 12h cooldown, recipients = group.members.
- Gated on opt-in pref `prefs.notifications.new_post`.
- Called fire-and-forget from `ActivityParser/handlers/Create/index.js` `createNotifications`.

**Default flipped ON:** `server/schema/User.js` `new_post` default `false`→`true`; backfilled existing users (`db.users.updateMany({"prefs.notifications.new_post":{$ne:true}},...)` → 11 on kwln.social, 54 on kwln.city).

**Remote followed-user posts — DONE 2026-07-28.** The Create-handler nudge only covered LOCAL posts; a followed REMOTE user's post fans out via a separate path (`methods/federation/pullFromRemote.js`, which uses the origin's recipients map — we can't compute remote circle memberships). Added `notifyFeedFanOut(pairs)` export in notifyFeedActivity.js and call it from pullFromRemote for users who got NEW per-user fan-out rows (keyed off the bulkWrite's `upsertedIds`; re-pulls with `fanOutCount:0` don't re-nudge). GOTCHA that cost hours: `pullFromRemote` runs in the **worker-pull** container — deploying only `app` left it stale and silently un-nudging (see [[project_production_servers]] worker-redeploy note). enqueueFanOut is LOCAL-only; the two fan-out paths (local Create → enqueueFanOut, remote → pullFromRemote) are genuinely separate.

Notification type is `new_post`. Web has the pref toggle (ProfilePage); **mobile toggle still TODO** (needs next build; part of the #78 preferences screen). Deferred: remote followed-user posts triggering the feed nudge (needs federation-pull integration, [[project_federation_pull_architecture]]). Delivery is still 60s foreground polling ([[project_notification_delivery_options]]).
