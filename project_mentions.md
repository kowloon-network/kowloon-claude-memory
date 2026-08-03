---
name: project_mentions
description: "@user@domain tagging in posts/replies (issue #77) — server-side linkify (anyone) + notify (local only)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Issue #77: tag community members by writing `@username@domain` inline in a post OR reply. Built + deployed to both servers 2026-07-27. **Server-only feature** — no editor work, so it covers web + mobile at once (deliberately avoided the tentap autocomplete rabbit hole; see [[project_mobile_composer_editor]]). Josh's decisions: type the handle (no picker); **link anyone (local or remote), notify local only**; "test it and see, remove if spammy."

- `server/methods/mentions/linkify.js` (PURE — no imports, so schema can use it without a cycle): `extractMentions(content)` + `linkifyMentions(content)`. Leading-boundary regex `/(^|[^\w@/])@([\w.-]+)@(domain)/g` skips email addresses. Linkifies ANY valid handle to `https://<domain>/users/@username@domain` — the form BOTH the web router and the mobile app route to a profile (mobile `parseKowloonUrl` matches `/users/(@..@..)`, opens in-app via `openKowloonLink` from [[project_kowloon_links_in_app]]). NOT the bare-username actorId form.
- `server/methods/mentions/notify.js`: notifies LOCAL tagged users only. New `mention` notification type (Notification enum) + `prefs.notifications.mention` pref (default true — closed subschema, needed the field). Skips self (createNotification also guards), dedups per `groupKey: mention:<objectId>:<recipientId>` so edits don't re-notify.
- Linkify wired into `Post.js` + `Reply.js` pre-save (rewrites the rendered `body`; stored `source.content` stays raw/editable). Notify wired fire-and-forget into the Create handler (`createNotifications`, Posts only) and the Reply handler.

Known v1 gaps (acceptable for the test): handles inside code spans also linkify; a REMOTE user mentioning a LOCAL user does NOT notify (notify only fires on local Create/Reply, not federated inbound — a federation follow-on). Mobile pref toggle rides with the #78 preferences screen; pref works server-side now. Non-Kowloon (Mastodon) remote links are best-effort. Related: [[project_new_post_notifications]], [[project_attachment_serializers]].
