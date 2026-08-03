---
name: following-circle-gotcha
description: "Following circle is type Circle + self-addressed (private); it only lists for authenticated owner requests, so a stale session hides it."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

The auto-created **Following circle** is created in `schema/User.js` as
`type: "Circle"` (NOT "System" like Groups/AllFollowing/Blocked/Muted) and
addressed `to: <ownerId>` (private). This is intentional and load-bearing:

- `GET /users/:id/circles` (`routes/users/circles.js`) filters `type: "Circle"`,
  so Following appears in the user's circle list *because* it's type "Circle".
  Do NOT "fix" it to `type: "System"` to match the design doc — that would make
  it vanish from every circle list.
- Because it's self-addressed, the endpoint returns it ONLY on the owner path
  (`user.id === userId`). The non-owner path filters `to` to `[@public,@domain]`,
  which excludes it. So an **unauthenticated / stale-session** request gets an
  empty list, and it looks like "my account has no Following circle."

Debugged 2026-07-20: jzellis's Following circle was missing from the mobile
feed/audience selectors in Expo Go after a kwln.social reinstall. Root cause was
a stale session (auth not landing) — **logging out and back in fixed it**. Not a
bug. Confirmed it always worked in the EAS build.

Mobile identifies Following for pinning (issue #37) by its self-address
(`c.to === account.id`, name fallback) in `src/lib/orderUserCircles.js` — NOT by
type. Related: [[feedback-no-follow-notifications]], [[project-codebase-gotchas]].
