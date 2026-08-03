---
name: project_image_orientation_fix
description: "How Kowloon fixes sideways photo uploads across mobile, web, and server (EXIF orientation normalization)."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Photos uploaded sideways because the OS picker/browser strips the EXIF Orientation tag while leaving pixels rotated, so the server has nothing to auto-orient from. Fixed at three layers (settled 2026-07-22):

- **Server** (`server/routes/files/upload.js`): bakes orientation when `(meta.orientation ?? 1) > 1` via `sharp(buffer).rotate()` (auto-orient + strip tag) + `resize(fit:'inside')`. Deployed. This ALONE fixes desktop web, because a browser `<input type=file>` sends the original file with its EXIF tag intact. Verified: sharp turns an Orientation=6 image into upright 200x400 with the tag gone.
- **Mobile** (`mobile/src/lib/normalizeImage.js`, wired into `mobile/src/lib/uploadFile.js` — the single upload chokepoint): uses `expo-image-manipulator` (added 2026-07-22, bundled in Expo Go so testable without a build) to re-encode JPEG/HEIC upright + cap longest side at 2048. Needed because the native picker strips the tag before the server sees it. RELIES on the manipulator's native decoder auto-orienting on load — VERIFY on-device with a known-sideways photo before trusting; installed apps need a fresh build (native module, not OTA-able).
- **Web** (`frontend/src/lib/normalizeImage.js`, wraps `client.files.upload` once in `frontend/src/lib/client.js`): DETERMINISTIC — parses the EXIF Orientation tag itself and applies the exact canvas transform to raw pixels (`imageOrientation:'none'`), so it can't double-rotate or depend on browser auto-orient. Closes the mobile-web-browser gap (some mobile browsers strip the tag like the native picker). Ships via normal frontend deploy, no app build.

All layers: JPEG/HEIC only; GIF/PNG/WebP/video/audio pass through untouched (avoid flattening GIFs / wrecking PNG alpha). See [[feedback_no_builds_without_permission]].

GOTCHA exposed while fixing Diana's images: `server/routes/files/serve.js` sends `File.size` as `Content-Length`. If the stored object and `File.size` ever drift (e.g. a script rewrites the object without updating size), clients truncate the download and show a gray tail. Normal uploads set size correctly; only hand-edits bite. Optional hardening: fall back to the real object length in serve.js.
