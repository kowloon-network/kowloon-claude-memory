---
name: no-builds-without-permission
description: Never trigger an EAS/Android/iOS build unless Josh explicitly asks in-turn. Build quota is limited (~15/mo per platform).
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Never run `eas build` (or otherwise trigger an Android/iOS build) unless Josh
explicitly tells me to in that turn. Prior approval to build once does NOT carry
forward.

**Why:** the EAS plan allows only ~15 Android + 15 iOS builds/month, and several
have already been spent (including wasted errored ones). Builds are a scarce,
metered resource.

**How to apply:** Batch native-dependency changes and verify everything possible
in Expo Go first (most JS changes hot-reload; server changes deploy via Docker CI
per [[project-production-servers]]). When a batch genuinely needs a fresh APK to
test native bits (e.g. expo-image, expo-media-library), say so and WAIT for Josh
to ask for the build. Related: [[project-mobile-share-intake-todo]] (EAS gotchas).
