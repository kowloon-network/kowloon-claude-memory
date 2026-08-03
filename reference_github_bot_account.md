---
name: reference_github_bot_account
description: The kowloon-claude GitHub bot account — how to authenticate gh and attribute commits to it instead of Josh.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Josh set up a dedicated GitHub bot account (2026-07-23) so my activity isn't attributed to him.

- **Account:** `kowloon-claude` (user id `308340492`), collaborator with **push** on all four repos (kowloon, kowloon-mobile, kowloon-frontend, kowloon-client).
- **Token:** a **classic** PAT (`ghp_…`, `repo` scope) stored at `~/.config/gh-kowloon-bot-token` (chmod 600, outside any repo). The `!export GH_TOKEN=...` prefix does NOT persist into Bash-tool subshells, so read it per-block:
  `export GH_TOKEN=$(cat ~/.config/gh-kowloon-bot-token)` before any `gh` / API call.
  MUST be a classic token: a fine-grained PAT can't write to repos owned by a
  DIFFERENT personal account (jzellis's) even as an outside collaborator — it
  403s "Resource not accessible by personal access token" regardless of scopes.
  Verified working 2026-07-23 (posted + deleted a test comment as kowloon-claude).
- **Commit attribution:** remotes are SSH (`git@github.com:...`), so pushes authenticate with Josh's SSH key — but GitHub attributes commits by AUTHOR EMAIL. Author commits as the bot WITHOUT touching Josh's git config (he commits to these repos too):
  `git -c user.name="kowloon-claude" -c user.email="308340492+kowloon-claude@users.noreply.github.com" commit ...`
- **gh (issues/labels/PRs):** run with `GH_TOKEN` exported from the file above; it posts as kowloon-claude.

**STATUS: fully working as of 2026-07-23** — comments, labels, and (by author email) commits all attribute to kowloon-claude. The earlier fine-grained token was a dead end (see above); the classic `repo`-scoped token resolved it.

Never put the token VALUE in memory or commit it. See [[feedback_pending_label_convention]] for the label/comment workflow this enables.
