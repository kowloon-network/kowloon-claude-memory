---
name: reaction-model
description: Reactions are ONE-per-user-per-target. A React activity sets/replaces it; an empty emoji clears it. Server exposes myReact + reactCounts.
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Reactions (settled 2026-07-18, replacing the old per-(user,target,emoji) upsert):

**One reaction per user per target.** A `React` activity SETS the user's single
reaction (add or replace); a React with an empty/omitted emoji CLEARS it. There
is no separate client un-react call — `client.activities.react({postId, emoji})`
with a falsy emoji clears (the client omits the object `type` field so the
server's emoji fallback doesn't read "React").

**Server (`ActivityParser/handlers/React/index.js`):** collapses any old stacked
reactions on write, recomputes `reactCount`/`reactPreview`/`reactSummary` from
the collection (not +1/-1), notifies the author ONLY on a brand-new reaction,
and federates the React (peers apply the same set/replace/clear). `Undo{React}`
from remotes is handled in the Undo handler (`syncTargetReactState` is exported
from the React handler). Federation still speaks Undo at the S2S edge.

**Reads:** `GET /posts/:id` returns `myReact` (the viewer's emoji or null) and
`reactCounts` ([{emoji,count}] for the reacts bar). `GET /posts` (feed) returns
`myReact` per card.

**Client:** ReactButton shows the viewer's own emoji once reacted (tap current =
remove, different = replace in one tap). ReactsBar is a read-only per-emoji count
summary on the post page (display-only per Josh; not tappable). Related:
[[project-undo-react-todo]] (now resolved), [[feedback-no-follow-notifications]].
