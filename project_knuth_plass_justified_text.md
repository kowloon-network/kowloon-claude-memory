---
name: project-knuth-plass-justified-text
description: "Future idea: TeX-quality justified body text (Knuth-Plass) for post/page reading surfaces; web-feasible, mobile needs custom work"
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Idea parked 2026-07-21 (not being built now): render long-form post/page body text with **Knuth–Plass line-breaking** — the whole-paragraph justification algorithm TeX uses (plus TeX hyphenation, optical margins, micro-tracking) — for publication-grade justified prose. Fits Josh's editorial aesthetic ([[feedback_design_aesthetic]]).

**Reference implementation:** `justif` — https://github.com/lyallcooper/justif (TS, browser progressive-enhancement; hands it DOM `<p>`s, it re-lays-out the lines).

**Feasibility split (the key constraint):**
- **Web frontend: doable, ~1-2 days.** Bodies render as real HTML via `dangerouslySetInnerHTML` (`frontend/src/components/posts/PostBody.jsx`, `frontend/src/pages/PageDetailPage.jsx`), so justif's target DOM exists. Wire it in a `useEffect` (call `justify(container.querySelectorAll('p'))`, keep the controller, dispose on unmount, re-run when reactive typography font/size changes), lazy-load the hyphenation pattern (bulk of the weight). Scope to article/page reading surfaces only — NOT feed cards or short Notes (narrow columns justify badly).
- **Mobile: not with justif.** RN renders HTML through `react-native-render-html` → native `<Text>`/`<View>`; no DOM, no Canvas, no `document.fonts`. justif can't attach; even `justif/core` needs per-glyph measurement RN doesn't expose, and RN's native text engine can't consume Knuth-Plass breaks (only basic iOS `textAlign:'justify'`). A cross-platform version = reimplement Knuth-Plass ourselves + custom line-by-line rendering. Big lift, deferred.

Net: if ever pursued, most realistic first step is **web-only** on the reading surfaces; true web+mobile parity requires our own implementation. Don't chase mobile parity with an off-the-shelf lib — there isn't one ([[project_mobile_strategy]]).
