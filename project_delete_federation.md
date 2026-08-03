---
name: project_delete_federation
description: How deleting a federated post propagates — soft-delete/tombstone + push-by-audience + pull serves recent tombstones so pullers self-heal (built 2026-07-28)
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

How Kowloon handles deleting content that has already federated. Settled/built 2026-07-28.

**Local delete = soft-delete/tombstone, not hard delete.** `ActivityParser/handlers/Delete/index.js` sets `deletedAt` + `type: "Tombstone"` (Users get `active:false` instead) and hard-deletes the origin's own `FeedItems` via `purgeFeedItems` (so the FeedItem is GONE, the object record is tombstoned). Owner-or-admin authorized.

**Two propagation channels (both needed — one doesn't cover the other):**
1. **Push, by audience** — on delete, `getFederationTargets` (ActivityParser/handlers/utils/getFederationTargets.js) resolves targets from the post's `to`: `@public`→followers, `@domain`→that domain, `circle:`→members, `group:`→members, and the Delete activity is pushed to them. A receiving server normalizes inbound `Tombstone`→`Delete` (normalizeInboundActivity) and runs its Delete handler → purges its cached copy. Covers addressed/pushed content.
2. **Pull self-heal (built 2026-07-28)** — the batch-pull `GET /outbox` (`from`+`to` mode) ALWAYS excludes tombstoned items from `items`, so pullers never learned about deletions via pull. Now the incremental (since) response also returns `tombstones` = ids of the pulled from-authors'/servers' Posts with `deletedAt >= since`. `pullFromRemote` reads `data.tombstones` and purges them from local `FeedItems` + `FeedFanOut` and tombstones any local Post shadow. This plugs the **firehose-subscriber gap**: a server that pulled a `@public` post via server-subscription (not tracked as a follower) wouldn't get the push, but self-heals on its next pull.

**Retention: 30 days, reuses the existing GC.** `methods/gc/index.js` `startGCWorker` (started from `index.js`) hard-deletes soft-deleted content older than `gcRetentionDays` (default 30), purging the object + its files + FeedFanOut. So tombstones are servable for 30 days, then gone — a puller absent longer than that won't hear about those deletions (acceptable edge). No separate retention config was added.

**Scope:** the pull-tombstone covers **Posts**. Replies dodge deletion-propagation entirely via the on-demand model (parent host is the single source of truth — [[project_reply_tombstone_refanout_todo]]). Pages are viewed on-demand (a deleted remote page 404s on next view). Extending pull-tombstones to Pages/Replies is a possible future symmetry, not needed now. Federation background: [[project_federation_pull_architecture]]. Deploy note: `pullFromRemote` runs in **worker-pull**, the outbox route in **app** — redeploy both ([[project_production_servers]]).
