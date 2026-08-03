---
name: project-production-servers
description: "Production server hostnames, SSH access, docker-compose location, deploy procedure, and seed credentials"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

## Production servers

| Server | Hostname | Notes |
|--------|----------|-------|
| kwln.social | kwln.social | Primary Kowloon server (full stack). Box IPv4 172.236.8.71, 15GB RAM. |
| kwln.city | kwln.city | Second Kowloon server, **co-hosted on the kwln.social box** (added 2026-07-20). For Josh's FB friends. |
| kowloon.network | kowloon.network | NOT a Kowloon server — a tiny (~1GB) box running only the Astro static site (`kowloon-network-web` container). Don't try to deploy the app stack here. |

## Co-hosting (kwln.city on the kwln.social box)

A second server on one box via compose-project isolation, wired 2026-07-20:
- kwln.city installed in `~/kowloon-city/` (its own compose project → own Mongo/MinIO/volumes, e.g. `kowloon-city_mongo_data`; physically separate from kwln.social's `kowloon_mongo_data`).
- Generated in the installer's **co-host mode**: no bundled Caddy, no host ports; its `app` joins the external `kowloon-edge` docker network with alias `app-kwln-city`.
- kwln.social's existing Caddy is the shared front door: it's attached to `kowloon-edge` and has a `kwln.city { reverse_proxy app-kwln-city:3000 }` block. Auto-TLS via Let's Encrypt per domain.
- Manage kwln.city: `cd ~/kowloon-city && docker compose ...`. Its own admin account (jzellis / jzellis@gmail.com).
- GOTCHA: `docker compose up -d` in `~/kowloon` recreates whatever image is newest locally — a `pull` in the kwln.city stack can bump kwln.social's app image as a side effect (data is safe; volumes persist). Backups of the wired files: `~/kowloon/*.bak.cohost`.
- NAME-COLLISION FIX (2026-07-20): the live kwln.city stack was deployed with its app service literally named `app`, which claimed the `app` DNS name on `kowloon-edge` and made kwln.social start serving kwln.city. Fix on the live box: kwln.social's Caddyfile vhost targets its container `kowloon-app-1:3000` (not bare `app`) — so `~/kowloon/Caddyfile` intentionally diverges from the installer default. The installer generator was fixed to name co-host apps `app-<slug>` (never `app`), so future co-hosts don't need this workaround. If kwln.city is ever redeployed with the new image, its app service becomes `app-kwln-city` and kwln.social's vhost could go back to `app`.
- To add server #3: installer into a fresh dir, tick co-host; the shared Caddy just needs another vhost block + it's already on `kowloon-edge`. See `[[project-install-frontend]]`.

## SSH access

SSH as `jzellis` with the local ed25519 key — no password needed:

```bash
ssh jzellis@kwln.social
ssh jzellis@kowloon.network
```

Root login is disabled (publickey only, no password).

## Docker compose location

`~/kowloon/docker-compose.yml` on both servers.

Services: `app`, `worker-feed`, `worker-pull`, `worker-outbox`, `worker-media`, `worker-backup`, `caddy`, `mongo`, `minio`.

## Deploy procedure

CI (GitHub Actions, repo `jzellis/kowloon`) builds and pushes `ghcr.io/jzellis/kowloon:latest` (server + frontend bundled) on every push to main. Frontend pushes dispatch the same build via `trigger-build.yml`; if that flakes on a transient GitHub 5xx (or the build itself fails on a Docker Hub `node:22-alpine` pull timeout), re-run `gh workflow run docker.yml --repo jzellis/kowloon --ref main` or `gh run rerun <id> --failed`. Servers do NOT auto-pull — SSH in and restart.

**Both Kowloon servers run on the ONE kwln.social box** (`ssh jzellis@kwln.social`):
- kwln.social → `~/kowloon`
- kwln.city → `~/kowloon-city`  (NOT on kowloon.network — that host is only the Astro static site, deployed separately)

Deploy a code change (2026-07-27, the exact commands I use):
```bash
ssh jzellis@kwln.social 'cd ~/kowloon      && docker compose pull app && docker compose up -d app'
ssh jzellis@kwln.social 'cd ~/kowloon-city && docker compose pull app && docker compose up -d app'
```
**CRITICAL: redeploy `worker-*` too when the change touches ANY code the workers run** — not just app-served routes. Federation/fan-out/scheduled work runs in workers: `pullFromRemote` (remote pulls + the #75 feed nudge) runs in **worker-pull**, feed fan-out in worker-feed, outbox federation in worker-outbox, media in worker-media, backups in worker-backup. Deploying only `app` leaves the workers on the OLD image — symptom: the app-container behavior is correct (e.g. getTimeline's on-demand pull nudges) but the scheduled/worker path is stale (silently no nudge). Bit me 2026-07-28 (#75 worker-pull was 5 days stale). Full redeploy of everything:
```bash
for d in kowloon kowloon-city; do
  ssh jzellis@kwln.social "cd ~/$d && docker compose pull app worker-feed worker-pull worker-outbox worker-media worker-backup && docker compose up -d app worker-feed worker-pull worker-outbox worker-media worker-backup"
done
```
Verify a worker actually got the code: `docker exec kowloon-worker-pull-1 grep -c <newSymbol> <file>`. Always verify with a live curl against the actual endpoint the client hits (the container reports `Up ... seconds` immediately but takes a few seconds to serve).

## Seed credentials

All seed users (created by `seeder/seed.js`) share the same password: `Kowloon2026!`

The seeder lives at `~/Projects/kowloon/seeder/seed.js` (separate from `server/scripts/`).

Group owners on both servers:
- Small Press & Zines → inkandpaper
- Jazz & Improvisation → nightwriter
- Slow Web Collective → cityhacker
- Field Recording Society → recordhead
- Letterpress & Print → designthink

**Why:** Needed repeatedly for API scripts (e.g. seed-group-images.js) that must log in as group owners.

**How to apply:** When running API scripts against production, use `--password=Kowloon2026!` and look up the group's `actorId` to find the right username to log in as.
