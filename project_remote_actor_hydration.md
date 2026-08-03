---
name: remote-actor-hydration-shape
description: Hydrating a remote AP actor into the local User schema hits shape mismatches (publicKey object vs PEM string); normalize fields before upsert.
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

When `methods/core/getObjectById.js` runs with `hydrateRemoteIntoDB: true`, it
upserts a fetched **remote AP actor** straight into the local `User` schema via
`findOneAndUpdate({ id }, { ...payload, ... })`. ActivityPub actors use nested
object shapes for fields that Kowloon's `User` schema stores as **scalars**, so
spreading the raw payload throws a Mongoose CastError.

- Confirmed + fixed (2026-07-15): `publicKey`. AP sends `{ id, owner, publicKeyPem }`;
  `User.publicKey` is a PEM **String** (`schema/User.js`). Now normalized to
  `publicKey.publicKeyPem` before the upsert.
- Confirmed + fixed (2026-07-22): **identity fields were never mapped.** The upsert
  keyed on `payload.id` (the actor URL) and spread the raw actor, so a hydrated
  remote user got `id = https://host/users/name` (should be the Kowloon handle
  `@name@domain`), `username = undefined`, `actorId = undefined`. This 404'd remote
  profile views (`GET /users/:id` is local-only; the app now falls back to `/lookup`
  in `client/src/feed/index.js#getUser`). Fix: map `id=idStr`,
  `username=preferredUsername||handle.username`, `actorId=payload.id`,
  `objectType='User'`, and key the upsert on `{ id: idStr }`. This is the start of the
  real mapper called for below.
- This path is ONLY hit by `hydrateRemoteIntoDB: true`, i.e. `GET /lookup` (and
  the old `/users/lookup`). Everything else uses lightweight member subdocs
  (`methods/parse/toMember.js`) or the FeedItems `actor` subdoc, so the bug was
  latent for a long time — federation "worked" without ever exercising it.

**Watch list (not yet fixed — only `publicKey` errored so far):** other AP-object
fields could bite the same way if a remote actor includes them and the local
schema wants a scalar — notably `icon` (AP `{ type:"Image", url }` vs the string
Kowloon expects), possibly `image`, `endpoints`, `attachment`. If a new cast error
appears on remote-user hydration, normalize that field too.

**Why:** Subtle, latent, and only reachable through one endpoint — easy to
rediscover the hard way. The AP↔Kowloon impedance mismatch is the root, not any
single field.

**How to apply:** If you touch remote-actor hydration or add fields to `User`,
the robust fix is a real AP-actor→User **mapper** instead of spreading the raw
payload. Until then, normalize offending object-shaped fields to scalars right
before the `findOneAndUpdate` upsert. See [[federation-pull-architecture]] and
[[codebase-gotchas]].
