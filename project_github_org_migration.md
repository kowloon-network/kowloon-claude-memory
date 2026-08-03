---
name: project_github_org_migration
description: "Plan + progress for moving the Kowloon repos into a GitHub org (target: kowloon-network) and consolidating issues via an org Project. CI prep done 2026-08-03."
metadata: 
  node_type: memory
  type: project
  originSessionId: 301f4471-2e43-49d8-b18f-9537cb26f3e9
---

Moving the four Kowloon repos from `jzellis/*` into a GitHub **organization** and
setting up an org-level **Project (Projects v2)** to get one cross-repo issue
tracker (issues stay in their repos; the Project is the unified board — do NOT
physically merge issues across repos).

**Org name:** `kowloon-network` (matches the domain; verified available 2026-08-03).
Bare `kowloon` is taken by a user. Create at https://github.com/organizations/plan (Free).

**Repos to transfer:** `jzellis/kowloon` (server), `kowloon-client`, `kowloon-frontend`,
`kowloon-mobile` (+ optionally kowloon-hosting / the website). GitHub preserves
issues/PRs/stars/history and auto-redirects old URLs.

**DONE — safe CI prep (2026-08-03), verified green in CI:**
- `server/.github/workflows/docker.yml`: the kowloon-frontend + kowloon-client
  checkouts now use `${{ github.repository_owner }}/…` (commit `2c863050`).
- `frontend/.github/workflows/trigger-build.yml`: dispatch uses `context.repo.owner`
  instead of hardcoded `owner: 'jzellis'` (commit `9b7c123`).
- The build image path already used `ghcr.io/${{ github.repository_owner }}/kowloon`,
  so it auto-follows the transfer. Both still resolve to `jzellis` until transfer.

**CUTOVER DONE 2026-08-03:** all 7 repos transferred to `kowloon-network` (redirects live); 4 local remotes repointed; org CI build verified green; `ghcr.io/kowloon-network/kowloon` published + made public (needed an org-level toggle first: Org Settings → Packages → "Package creation" → allow **Public**, which is OFF by default on new orgs — the per-package Public radio is greyed out until then); BOTH boxes' docker-compose (`~/kowloon` + `~/kowloon-city`, 6 image refs each) repointed to the org image, pulled, `up -d`, verified 200 on kwln.social + kwln.city, no container left on the jzellis image; `mobile/package.json` clone URL updated. `docker-compose.yml.bak.orgmig` backups left on both boxes. Repo/CI is [[reference_github_bot_account]] `repo`-scope only — it CANNOT read/write GHCR packages or Projects v2 (those need package/project scopes), so package-visibility + org-Project creation are Josh-only.

**STILL OPEN (non-blocking):** (1) `kowloon-setup` package still private — only the self-host installer pulls it, so make it public when convenient (same package-visibility flow). (2) Org-level Project (Projects v2) to unify issues across repos — Josh creates it (bot lacks project scope): Org → Projects → New project → add all repos.

**REMAINING — cutover (superseded by CUTOVER DONE above; kept for reference):**
1. Josh: create the org, add the `kowloon-claude` bot with write access, re-add
   secrets (`DISPATCH_TOKEN` on frontend, GHCR/EAS as needed), transfer the 4 repos.
2. Push once so the org publishes `ghcr.io/kowloon-network/kowloon:latest`; make it
   public (or grant the boxes' pull token).
3. **Update production docker-compose on BOTH boxes** — `ghcr.io/jzellis/kowloon`
   → `ghcr.io/kowloon-network/kowloon` (6 image refs each in `~/kowloon` AND
   `~/kowloon-city` on kwln.social), then `docker compose pull app && up -d app`.
   See [[project_production_servers]].
4. `mobile/package.json` `eas-build-pre-install` clones `github.com/jzellis/kowloon-client`
   — update to the org (or rely on GitHub's redirect; low urgency).
5. Repoint the 4 local git remotes (redirects cover it, but cleaner).
6. Create the org Project and add all repos.

Only ~5 hardcoded-ref edits total; the rest is Josh's interactive GitHub work.
Related: [[reference_github_bot_account]], [[feedback_dev_workflow]].
