---
name: website-plan
description: "Plan for kowloon.network — the open-source app's public site (Astro). Separate future paid-hosting platform (kowloon-hosting) deferred."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Turning **kowloon.network** from a test community server into the open-source
app's home on the web (kwln.social stays the flagship community server; test
data on kowloon.network not migrated). Decided 2026-07-18.

**Repos (both private, created):** `kowloon-network` (the Astro site) and
`kowloon-hosting` (empty placeholder for the future Stripe-billed hosting
platform — part 2, NOT being built now).

**Stack:** Astro (static, content-first). **Starlight** for `/docs` (themed to
Klein-blue/editorial). Custom Astro pages for home/about/faq. Blog/devlog IN for
v1. React islands only where interactive (waitlist form). Brand tokens lifted
from `kowloon-frontend`: Klein blue #002FA7, Lora + Inter, hard edges, hex mark.

**Launch surface:** self-hosting/install guide + docs/tutorials + waitlist; APK
link optional/toggleable; NO public web-client link yet.

**Waitlist + devlog newsletter:** **Kit (ConvertKit)** — chosen 2026-07-18 for
its free tier up to 10k subscribers (Buttondown's ~100 was too low). Managed 3rd-
party (NOT self-hosted Listmonk); site just needs an embed snippet, swappable.

**IA:** Home (pitch: circles-not-followers, no algorithm, own-your-data) -> Docs
(self-hosting first, then Using Kowloon, then Dev/API) -> FAQ -> About -> Blog ->
Hosting (teaser + waitlist now; real platform later).

**Deploy:** static build -> nginx on the kowloon.network box via the existing
Docker CI flow (build -> ghcr image -> compose pull+up), like [[project-production-servers]].

**Content:** I draft from CLAUDE.md/READMEs + Joplin design docs + installer flow;
Josh edits for voice. First-priority pieces: self-hosting guide + home pitch.

**STATUS (2026-07-18):** Phase 1 done — kowloon.network is LIVE with a branded
coming-soon page (hex mark on Klein blue), HTTPS. Repo `kowloon-network`
scaffolded (Astro 7, local at ~/Projects/kowloon-network) + pushed. Old test
Kowloon stack on the box torn down (`docker compose down`, volumes KEPT, config
archived to `~/kowloon.old`). **Deploy pipeline (RESOLVED):** GHCR container package `kowloon-network` was made
PUBLIC (repo stays private — image is just the public site; note package
visibility != repo visibility, that tripped us up). Box runs
`ghcr.io/jzellis/kowloon-network:latest` (Dockerfile: Astro build -> Caddy static
serve, auto-HTTPS) via compose at `~/kowloon-network/docker-compose.yml`.
**Deploy flow = same as the Kowloon server:** push to main -> CI builds+pushes the
image -> on the box `cd ~/kowloon-network && docker compose pull && docker compose up -d`.
(Bootstrap leftovers `~/kowloon-network/site` + `Caddyfile` are now unused/harmless.)

**HOME PAGE (shipped, live):** Astro home at src/pages/index.astro — full-bleed
splash illustration directly under a sticky top bar, **black page background**
(Josh's "for now" call; brand tokens in src/layouts/Base.astro), wordmark +
tagline + waitlist CTA, three principles (circles/no-algorithm/own-your-words),
waitlist section, footer. Copy is my draft (Josh edits). Splash is a PLACEHOLDER
(cheesy AI "FIX ME SHAN" cyberpunk image) at src/assets/splash.png, routed
through astro:assets (<Image>, responsive webp) — swap that one file for Shan's
real art. Local repo: /home/jzellis/Projects/kowloon-network.

**NEXT STEPS (queued):** (a) wire the waitlist CTA (currently href="#") to **Kit**
— needs Josh to create a Kit account + waitlist form, then give me the embed/form
id; (b) add **Keystatic** CMS (browser editor, commits to repo) once there's
content to manage; (c) start the **self-hosting docs** via Starlight; (d) swap in
Shan's splash art when it lands.

**Phasing:** (1) scaffold + brand + prove deploy pipeline (hello-world live) [DONE] ->
(2) home + waitlist + about -> (3) docs -> (4) FAQ + blog + SEO/OG polish ->
(5 later) hosting section. Related: [[reference-joplin-integration]].
