---
name: project_media_embeds
description: "Rich-media embeds (YouTube inline player): shared @kowloon/client embed registry + security model + server oEmbed preview enrichment. Built 2026-08-05."
metadata:
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Shared **embed registry** for rich media (YouTube first; Vimeo/SoundCloud/etc.
are one file each). Lives in **`@kowloon/client/src/embeds/`** — one provider file
(`providers/youtube.js`) exporting `{ name, hosts, match(url), embed(match) }`,
registered in `embeds/index.js`. `resolveEmbed(url)` is the single recognizer,
exported from the client's main index and the `@kowloon/client/embeds` subpath.
Add a provider = drop a file + add to `PROVIDERS` (like ActivityParser handlers).

Descriptor: `{ provider, mode:'inline'|'linkout', embedUrl, thumbnail, aspectRatio, allow, sandbox }`.

**Security model (Josh's hard requirement: no user can insert an iframe).**
- Post bodies already strip iframes on write: `server/schema/Post.js` `ALLOWED_TAGS`
  excludes iframe/script/object/embed, `disallowedTagsMode:"discard"`. Users can't
  persist one.
- The player is **derived from `post.href` at RENDER time** via `resolveEmbed`, never
  authored or stored as markup. The iframe `src` is built from a strictly-validated
  id (YouTube `[A-Za-z0-9_-]{11}`) against a fixed provider template — nothing
  user-supplied reaches the src. Re-derived every render, so nothing tamperable is
  persisted. Only allowlisted hosts embed; anything else stays a plain link card.
- Renderers use a **facade**: poster + play button, iframe/WebView mounts only on
  click (perf + no autoplay). `youtube-nocookie.com`, plus `sandbox`/`allow`.

**Web:** `frontend/src/components/posts/EmbedPlayer.jsx` in `PostBody`'s
featured-media slot. **Mobile:** `mobile/src/components/posts/EmbedPlayer.jsx` uses
`react-native-webview` (ALREADY a dep — no new native module) in `PostCard`'s Link
branch. Shipped in the 2026-08-05 tester build.

**Server `/preview` uses a SEPARATE oEmbed map, NOT the client registry.** Why: the
server image can't import `@kowloon/client` — `server/Dockerfile.prod` uses the
client only in stage 1 to build the frontend; the stage-2 server image never
installs it. So `routes/preview/index.js` has its own small `OEMBED_PROVIDERS`
list. This is a different concern anyway: **preview enrichment** vs **render**.
YouTube serves bot-generic OG data (title "YouTube", no thumbnail), so `/preview`
hits the provider's public **oEmbed** endpoint (no key) for the real title +
thumbnail before falling back to the generic scrape. Unifying server onto the
shared registry later would mean baking the client into the server image.

**Composer gotcha (fixed):** `PostComposer` auto-fills title/featured/body from
`/preview`. It must seed **once per URL** (`autoFilledHrefRef`) — the old
`!content.trim()` guard re-injected the quoted body on every debounced re-fetch,
so a user couldn't erase it (deletion left it empty → next fetch re-added it).

Related: [[project_kowloon_links_in_app]], [[project_web_app_parity]], [[project_attachment_serializers]].
