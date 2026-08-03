---
name: pending-label-convention
description: "On kowloon-mobile GitHub, the yellow \"Pending\" label means the issue is awaiting testing or action from Josh — not actionable by me."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

On the `kowloon-mobile` GitHub repo, the yellow **"Pending"** label (#fbca04,
created 2026-07-18) means the issue is **awaiting either testing or action from
Josh** — e.g. it needs a build before he can verify, or he has to do part of it
himself.

**How to apply:** Treat Pending-labeled issues as in Josh's court. Don't pick
them up, re-work them, or treat them as open work for me unless he explicitly
asks. When I finish work and it's waiting on him, that's the state the label
represents. Fits the existing flow where I comment + assign issues to him rather
than closing them ([[feedback-no-builds-without-permission]] is a common reason
an issue sits Pending: it needs a build he must request).

**Workflow Josh set 2026-07-23 — apply automatically going forward:**
- When I fix an issue, ADD the **`Pending`** label. Do NOT close it — Josh closes
  it after he tests. Leaving it open+Pending is how he tracks "fixed, needs my
  test."
- Do NOT write `Closes #X` / `Fixes #X` in commit messages for these — a same-repo
  push auto-closes the issue (bit us on kowloon-mobile #59/#60/#61/#63/#66). Use a
  plain `#X` reference instead. (Cross-repo `Closes #X`, e.g. a server-repo commit
  referencing a kowloon-mobile issue, does NOT auto-close — but still prefer plain
  `#X`.)
- Additionally add **`waiting-for-build`** (blue #1d76db, created 2026-07-23, in
  kowloon-mobile) when the fix needs a NEW mobile build before Josh can test it —
  i.e. a mobile-client change. Server/web fixes that the CURRENT app exercises get
  `Pending` only (testable now). This gives three buckets: Pending-only (test now),
  Pending+waiting-for-build (needs build), and untagged (not tackled).
- When I fix an issue, also **post a short comment summarizing what I did** (root
  cause + the fix, 1-3 sentences; note "testable now" vs "needs new build"). Asked
  for 2026-07-23; applies going forward, not retroactively to the 2026-07-23 batch
  (#51,#52,#53,#56,#57,#59,#60,#61,#62,#63,#64,#66).
- Also **assign the issue to jzellis** when it goes Pending (it's in his court to
  test). Confirmed 2026-07-23; the 12-issue batch above is already assigned to him.
  (`gh api` search index lags — verify assignment per-issue, not via `gh issue list
  --assignee`.) All of this runs as the bot ([[reference_github_bot_account]]).
