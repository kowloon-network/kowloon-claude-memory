---
name: audio-player
description: "Global single-instance audio player + queue for the mobile app (#83) — teal right-edge slide-out bar, background playback, lock-screen controls."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Mobile audio playback is unified through ONE global player (issue #83, built 2026-07-31). `src/lib/AudioPlayerProvider.jsx` — mounted at the root layout (`app/_layout.js`, inside PushProvider) — owns a single `expo-audio` `useAudioPlayer()` instance + a JS-managed queue. **Never create ad-hoc `useAudioPlayer`/audio sheets elsewhere**; route audio through `useAudioBar().requestTrack({ id, url, title })` so only one clip ever plays.

**Queue model** (refs are authoritative to avoid stale closures; a `useReducer` bump forces re-render): `requestTrack` starts fresh if idle, toggles if it's the current track, else shows a **Play now / Add to queue** prompt. `playNow` inserts after current + skips to it; `enqueue` appends; auto-advances on `didJustFinish`. Track swap = `player.replace({uri})` (not a new player). `seekBy(±15)`, `prev` (seek-0 if >3s in), `next`, `stop` (clears queue + lock screen).

**UI** — `AudioBar` renders a **right-edge slide-out**: a small **teal** (`bg-post-media` #009084, matches the Media icon) audio tab clings to the right edge (`Animated` translateX), **starts minimized** (less jarring — do NOT auto-expand on new track), tap to slide out the full controls (rw/prev/play-pause/next/ff + close + progress + queue counter). Vertical position fixed at ~38% down (clear of OS bars). Josh vetoed the floating bottom bar.

**Consumers wired to it:** `DiscoverMediaTile` (Discover/server media tiles — audio kind) and `AudioAttachment` (post audio). Video still opens fullscreen via `expo-video` in `DiscoverMediaTile` (PIP-video queue is a noted future follow-up in #83).

**Background playback** (needs a build — native config): `app.json` ios.infoPlist `UIBackgroundModes:["audio"]`; `setAudioModeAsync({ playsInSilentMode, shouldPlayInBackground:true, interruptionMode:"doNotMix" })`; on play `player.setActiveForLockScreen(true, {title, artist})` (drives the Android media-notification foreground service — the FOREGROUND_SERVICE[_MEDIA_PLAYBACK] perms were already in app.json), cleared on stop; `requestNotificationPermissionsAsync()` on mount. Keeps playing when backgrounded/locked with lock-screen controls; **force-close still stops it** (universal — process death). Related: [[recommendations-discover-architecture]] (media tiles), [[project-mobile-redesign-2026-07]].
