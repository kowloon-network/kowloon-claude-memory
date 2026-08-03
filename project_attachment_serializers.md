---
name: project_attachment_serializers
description: "Post attachment {url,mediaType,name,alt} objects are built in FOUR separate server places; add new fields to all of them"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

On the Kowloon server, a Post's `attachments` (stored as `[String]` File IDs) are resolved into `{url, mediaType, name, alt}` client objects in **four separate, duplicated builders** — there is no single serializer. If you add/change an attachment field you must edit all four or it silently works on some feeds and not others:

1. `server/methods/files/enrichAttachments.js` — used by `/users/:id/posts` and `/groups/:id/posts`.
2. `server/routes/posts/collection.js` — the main `/posts` feed + `/posts/server`.
3. `server/routes/posts/id.js` — single post `/posts/:id`.
4. `server/routes/circles/posts.js` — circle feeds `/circles/:id/posts`.

Gotcha that bit me 2026-07-27: `File.name` = title, `File.summary` = alt text. All four already `.select("... summary ...")` but only emitted `name` (with a `name: f.name ?? f.summary` fallback) and dropped `summary`. Added `alt: f.summary ?? ""` to all four (and stopped `name` falling back to summary) so the mobile full-screen viewer caption works. Fixing only #1 first meant `/posts` still lacked `alt` after deploy — verify against the actual endpoint the client hits.

Related: [[feedback_actor_canonical]], [[project_codebase_gotchas]].
