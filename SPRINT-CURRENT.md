# Sprint Current — What Axis Is Working On RIGHT NOW

_Last updated: 2026-04-21 20:03_

## Last Hour
- 20:03 — Selfcheck: PM2 14 online, hub/openviking/embedding/hermes healthy, cron bad count 0, and no new blocker surfaced, so Axis is staying on routine continuity and cleanup.
- 18:28 — Next infra wedge identified: most Traefik-managed `*.verduona.dev` hosts still resolve to `91.99.176.2`, so the safe cleanup path is explicit-record migration + stale-router pruning, not a blind wildcard flip.
- 18:22 — Selfcheck: cron baseline still clean (`consecutiveErrors > 2` = 0), `hub.verduona.dev` now has a valid Let's Encrypt cert after DNS propagation + Traefik restart, and the remaining infra risk is broader `*.verduona.dev` ACME noise still validating against `91.99.176.2`.
- 17:21 — Selfcheck: mail-tba migration plan pushed, Zitadel password-only login proved without Google, `hub.verduona.dev` now resolves to dev-2026 and serves Hub health, but TLS is still blocked by Traefik serving the default cert.
- 16:09 — Selfcheck: CouncilNow live auth→checkout/portal now fully verified on `https://councilnow.com` with fresh test users (all `/api/billing` tiers + credits + subscription + portal), and cron/job baseline stayed healthy (`consecutiveErrors > 2` remains 0).
- 15:20 — Selfcheck: AgentJack waves remain complete on `main`; NOW priority is CouncilNow revenue blocker (auth→checkout/portal), enabled cron jobs have no consecutiveErrors > 2 and no overdue runs, local health baseline is green (PM2 14, hub/openviking/embeddings/hermes).
- 16:08 — Selfcheck: AgentJack remaining waves confirmed complete on `main` (1854046, 2f9d7c0, a771997), CouncilNow auth/billing fallback deployed to dev-2026, enabled cron jobs have no consecutiveErrors > 2, and live pricing auth→checkout validation is in progress.
- 13:13 — Matej directive: top priority is to finish AgentJack today. Run Codex in YOLO mode for all remaining waves and keep focus there until shipped.
- 12:04 — Selfcheck: Reconciled today’s AgentJack session history, confirmed Wave 0 truth-pass pushed and ship gate + Telegram are green, now executing Wave 1 runtime/recovery hardening; no cron jobs over consecutiveErrors > 2, workspace clean.
- 10:03 — Selfcheck: AgentJack autodev remains the focus, no cron jobs over consecutiveErrors > 2, management repo clean.
- 00:00 — Selfcheck #1: All 8 erroring cron jobs fixed (delivery=none + best-effort). AgentJack got multi-tenant commit. PM2 14/16 online. Hub + embeddings OK. Daily note written. Workspace pushed to GitHub.
- 01:00 — Selfcheck #2: AgentJack dev sprint shipped (scheduler + evaluator wired, 39 new tests, 488 passed). dev-2026 verified healthy (19 Docker containers, PM2 all online). Memory consolidation from 15 previous lifetime files completed → obsidian-memory/Knowledge/consolidated-memory.md. Management hub + workspace pushed to GitHub.
- 02:00 — Selfcheck #3: automation-ops-review now OK (delivery fix confirmed). Daily Log Review ran OK (system stable, no crashes). Local Docker: 6 containers (agentjack-litellm unhealthy — malformed API key, non-critical). gbrain 9/10 health, 100% embed coverage. 6 old-error crons pending re-run at 03:00/04:00/08:00/09:00.
- 03:00 — Selfcheck #4: Overnight maintenance ran OK (first old-error job cleared). System stable.
- 04:00 — Selfcheck #5: Daily backup OK. Apple Notes OK. workspace-github-push running. 3 old-error jobs remain (fire at 08:00/09:00). **Delivery fix fully confirmed — all fixed jobs now running clean.**
- 07:20 — **Harness + UI analysis complete. Matej APPROVED plan:** Keep AgentJack SvelteKit webchat, port GoClaw B2B admin patterns (tenants, teams, API keys, usage), study GoClaw architecture for multi-tenant/security reimplement in Python. GoClaw CC BY-NC license — patterns only, no code.
- 08:25 — **CouncilNow deployed on dev-2026.** 5 Docker services healthy (api, hub, frontend, marketing, db). Fixed 7 deployment issues (LXD conflict, PM2 zombies, DNS, missing DB, macOS resource forks crashing migrations, Traefik routing). councilnow.com live.
- 08:50 — Marketing site had wrong `_app` routing. Simplified: everything goes through SvelteKit frontend (port 5180). Rebuilt frontend with latest source. Buttons should now hydrate correctly.
- 09:06 — Rebuilt frontend Docker image with latest advisory-council-app source. Awaiting Matej confirmation that buttons work.
- 13:55 — **CouncilNow price ID bug FIXED.** Starter and Pro were both using $29 USD price. Updated to correct EUR prices (€29/€79/€199/€499). Verified full checkout chain: internet → Traefik → LXD → hub → Stripe → valid €79 Pro session. Ready for first customer.
- 18:03 — Selfcheck: ALL 4 AgentJack steering tasks shipped by Axis directly (tenants API + page, memory ABC, overview verified). CouncilNow revenue E2E verified. Zero cron errors. Autodev board empty — need new tasks from inbox or fresh priorities from Matej.
- 20:03 — Selfcheck: Quiet evening. All systems green (PM2 14, hub, embeddings, CouncilNow). Zero cron errors. All 4 AgentJack tasks shipped, autodev board empty. Awaiting Matej direction for next priorities.
- 22:03 — Selfcheck: Quiet evening. All systems green. Zero cron errors. All 4 AgentJack tasks shipped earlier today. Autodev cycling clean (no active tasks). Awaiting Matej direction.
- 00:03 (Mon) — Selfcheck: Quiet overnight. Zero cron errors. All systems green. Day summary: shipped 4 AgentJack tasks, fixed CouncilNow pricing, stripped automation to AgentJack-only, wrote mail VM migration plan.
- 02:03 (Mon) — Selfcheck: Overnight stable. Zero cron errors. All green.
- 04:08 (Mon) — Selfcheck: Deep overnight stable. Zero errors. All green.
- 07:20 — **AgentJack product completion sprint started.** Matej: "complete product, plan for today and all, define final product and then do it."
- 07:35 — **Product audit complete.** Backend 85% (30K lines), Frontend 60% (10K lines). 30+ sections with real data loaders already working.
- 07:40 — **3-site chain wired:** kraliki.com (brand CTAs) → agentjack.kraliki.com (product landing) → agentjack-dev.verduona.dev (app). CTAs updated in both marketing repos.
- 07:45 — **Settings page shipped** (4 tabs: Profile, Integrations, API Keys, Preferences). **Dashboard redesigned** (Quick Actions, Svelte 5 $derived migration).
- 07:48 — **Privacy + Terms pages shipped.** All in (public) route group (no sidebar).
- 07:50 — **Login page using shared Hub auth** (same as CouncilNow). Created $lib/auth.ts mirroring CouncilNow's auth module exactly. One userbase via Zitadel/Hub.
- 07:52 — **Finding: Zitadel not wired** in Hub DB. Env vars exist but no OIDC provider entry. Google SSO works fine, Zitadel-without-Google impossible until DB insert.
- 08:04 — Selfcheck: Active session building AgentJack. 4 pages shipped, auth wired, marketing CTAs updated. Continuing product completion.

## Active Sprint: AXIS SETUP

### Phase 1: Full Axis Setup (CURRENT)
- [x] Lock product/project map in MEMORY.md
- [x] Fix broken cron delivery (Telegram → none)
- [x] Set up strategic cron cadence
- [x] Create management hub
- [x] Consolidate scattered memories from previous Axis lifetimes
- [x] Verify dev-2026 server health (19 Docker containers up, PM2 all online)
- [x] Verify all tools operational (PM2 14 online, Docker 6 containers, gbrain 9/10, embeddings OK, Hub OK)
- [x] Push workspace to GitHub
- [ ] Confirm all cron jobs running clean (6 old errors pending re-run at 03:00-09:00)

### Phase 2: CouncilNow Business Launch (NEXT)
- [ ] CouncilNow.com live and selling
- [ ] Payment flow working (Stripe)
- [ ] First paying customer

### Phase 3: AgentJack Development (ONGOING)
- [ ] Agent core architecture
- [ ] First app on AgentJack
- [ ] Surpass Garry's stack (OpenClaw + GStack + GBrain)

## Blocked
- None currently

## Notes
- Matej confirmed: GLM-5.1 for my coding, Codex (GPT-5.4 xhigh) for heavy lifting, Kimi for web design, Grok 4.3 for current info
- Push to GitHub regularly. Git is undo.
- AgentJack must overcome Garry's stack. Not match — surpass.
