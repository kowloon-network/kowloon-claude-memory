---
name: project_mobile_composer_editor
description: "How the mobile post composer body editor (10tap/tentap) is laid out for scroll; don't reintroduce dynamicHeight"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

The mobile composer (`mobile/app/compose.js`) body uses `@10play/tentap-editor`'s `RichText` (a WebView). Non-obvious traps, settled 2026-07-27 for issue #76 ("can't scroll body, dragging moves caret"):

- tentap ships the WebView with **`scrollEnabled: false`** by default and manages its own caret-scroll. So a fixed-height editor can't scroll its content at all, and the parent ScrollView steals the vertical pan (you just drag the caret). Fix: pass `scrollEnabled nestedScrollEnabled` to `RichText` (props are spread onto the WebView AFTER the default, so they override) to turn on real internal scroll.
- **Do NOT use `dynamicHeight: true`** to make it grow. Inside a ScrollView a WebView has no height ceiling, so the content-height measurement runs away to thousands of px (a ~5000px empty field). Rejected.
- Working layout: a **two-region flex column** inside the flex-1 content View. Fields (title/media/featured) live in a `ScrollView` with `{flexGrow:0, flexShrink:1}` (takes only the height it needs). The body editor is a separate sibling `View` with `{flex:1, minHeight:140}` so it fills the gap between the fields and the controls; it shrinks automatically when the keyboard opens, so it's always sized just above the toolbar.
- The formatting **Toolbar is pinned above the keyboard**, not inline: an absolutely-positioned `View` at `bottom: keyboardInset` (from `useKeyboardInset`), and DON'T pass `hidden` — tentap's Toolbar auto-hides (`display:none`) unless `isKeyboardUp && editorState.isFocused`, so it only shows while writing the body.
- Last-line clearance under the toolbar comes free: tentap injects ~44px bottom padding on `.ProseMirror` while the keyboard is up.

See [[project_mobile_app_scaffold]] and [[feedback_android_ripple_bg_repaint]].
