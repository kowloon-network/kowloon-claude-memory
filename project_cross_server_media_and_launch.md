---
name: project_cross_server_media_and_launch
description: "kwln.city went public (Facebook signup, open registration) 2026-07-22; cross-server media federation now works for public files; restricted remote media still TODO."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

**kwln.city is publicly launched** as of 2026-07-22 — Josh posted signup instructions on Facebook, real strangers are registering (open registration, `registrationIsOpen=true`, no invite needed). Co-hosted on the kwln.social box (see [[project_production_servers]]).

**Cross-server media federation** (built + deployed both servers 2026-07-22): a federated post references media as `file:<id>@<remote-domain>`; the receiving server had no File record so it dropped the attachment (all cross-server images/audio/video showed nothing). Fix: `methods/files/hydrateRemoteFile.js` caches remote File metadata from the origin's public `GET /files/:id/meta` at pull time (wired into `pullFromRemote.js`), and `fileServeUrl()` in `signedUrl.js` returns the origin URL for remote files. The 3 enrichment spots (`enrichAttachments.js`, `routes/posts/id.js`, `routes/circles/posts.js`) select `url` and use `fileServeUrl`. Mobile already has the `expo-audio`/`AudioAttachment` player in-build, so this lit up on existing builds without a rebuild.

**Still TODO (deferred, flagged at launch):** restricted (circle/server-only) cross-server media does NOT render — the origin's `/meta` and bytes are only anonymously fetchable for `@public` files, and cross-server signed URLs don't exist yet. `hydrateRemoteFile` skips non-public files by design. This is the next federation piece.

**Also still open:** kwln.city's server `name`/`description` settings are unset (blank on landing/app header) — Josh to provide wording. Don't invent them; `actor` is canonical, this is a Settings value.

Related: [[project_remote_actor_hydration]] (same pull-time caching pattern, for actors), [[project_federation_pull_architecture]].
