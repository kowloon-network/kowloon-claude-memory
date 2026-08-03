---
name: project_mobile_composer_media_ux
description: "Mobile composer media UX (2-up thumbnail grid, centered chooser, focus rules) + full-screen viewer title/alt caption"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Mobile composer (`mobile/app/compose.js`) UX overhaul, 2026-07-27 (branch `links-in-app`; video thumbs need the pending EAS build — native module). See [[project_mobile_composer_editor]] for the body scroll/toolbar architecture.

- **Open at top, focus only the body for Note.** `useEditorBridge({ autofocus: false })`; only `type === "Note"` calls `editor.focus()`. Titled types (Article/Media/Event/Link) focus their in-scroll title field, so the form opens with fields visible. (Josh rejected a pinned Article title — use the same above-the-body title field as the others.)
- **Merged media buttons + centered chooser modal.** One "Add media" button opens a **centered** Modal (Josh disliked the bottom-sheet) with Photo-or-Video / Audio / Cancel. Launch the picker via `setTimeout(pick, 180)` AFTER closing the modal (Android modal-dismiss vs picker-open race). Native `Alert.alert` 3-button layout can't be aligned on Android — that's why it's a custom Modal.
- **Attachments = 2-up thumbnail grid.** `flex-row flex-wrap`, tiles `w-1/2` `aspectRatio:1`, real thumbnails (video via `expo-video-thumbnails` `makeVideoThumb`, guarded require). Each tile has a top-right ✕ (`removeAttachment(i)`) and an ALT? badge. Empty state is a compact "+ Add media" button (NOT a giant tile). Tap a tile → centered editor Modal with larger preview + Title + Alt fields + Delete + Done. That modal shifts with the keyboard via `style={{paddingBottom: keyboardInset}}` (KeyboardAvoidingView doesn't move a centered modal on Android). No reordering for now.
- Upload passes `title: a.title?.trim() || a.name` and `summary: a.alt?.trim() || undefined`. On the File: `name` = title, `summary` = alt. Create handler drops title/alt from the payload — they're set on the File at UPLOAD time. See [[project_attachment_serializers]].

**Full-screen image viewer caption** (`src/components/ImageViewerProvider.jsx`): `open(items, index)` now accepts `{uri, title, alt}` objects (still back-compat with bare URL strings). Title overlays the top (reading font, text-shadowed for legibility over light images); alt overlays the bottom (white on `bg-black/60`); caption follows swipes via a state-tracked current index. `PostBody.jsx`/`PostCard.jsx` pass `title=att.name`, `alt=att.alt||att.summary`. Required the server to actually emit `alt` — [[project_attachment_serializers]].

**Multi-account switcher** (`src/components/UserMenu.jsx`): lists other stored accounts → `switchTo` via `setActiveAndPersist` + `router.replace('/feed')`; "Add account" → `router.push('/login')`. No logout/login needed. (The multi-account data model was there from day one — [[project_mobile_app_scaffold]].)
