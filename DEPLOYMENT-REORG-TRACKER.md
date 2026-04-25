# Deployment Reorganization Tracker

_Last updated: 2026-04-25 18:09 CET_

## Canonical infra plan (already exists)
- `/Users/matejhavlin/github/infra/MAIL-VM-MIGRATION-PLAN.md`
- `/Users/matejhavlin/github/infra/PRODUCTION-ARCHITECTURE.md`

Anchor statement confirmed in both docs:
- Static websites/marketing sites → **Cloudflare Pages**
- Non-mail runtimes/apps → **dev-2026**
- `mail-tba` Hetzner VM → **repurposed as DR standby for critical data** (not public app hosting)

## gstack lane: build-deploy-verify (mandatory)
For each deployment change, log:
- Build: result
- Deploy: result
- Verify: exact checks + outcomes
- Blockers: unresolved items + owner decision needed

## Two-line execution model (updated by founder directive)

### Line 1 — Mac command + top-level development (Matej-led)
- Mac is the control plane and top-level execution lane.
- Matej leads product direction and interactive development.
- TBA-One-PA runs orchestration/tracking with gstack + gbrain from Mac.

### Line 2 — dev-2026 autodevelopment + runtime hosting
- dev-2026 is the continuous autodevelopment lane.
- Axis/OpenClaw + Telegram lane lives on dev-2026 (separate from local TBA-One-PA control plane).
- dev-2026 hosts non-mail runtimes and selected infra.
- Application deployments on dev-2026 should follow **Ubuntu container** patterns.
- Canonical deployment reference: **CouncilNow app** on dev-2026.

## Product priority (locked)
1. **AgentJack first:** complete proprietary stable/reliable/secure B2B apex agent ASAP.
2. **Vertical apps second:** layer AgentJack vertical apps after core apex readiness is complete.
3. **CouncilNow lane (strategic):** councilnow.com is a funnel/demo/showcase and business tool; keep it separately deployable and positioned for both SaaS UX and API-based intelligent decision workflows.
4. **Ecosystem target:** kraliki.com as SaaS ecosystem surface, verduona.com as AI-driven product company surface.

## Hosting decomposition (locked)
- Static websites/marketing websites → **Cloudflare Pages/Websites**.
- Mail server decomposition:
  - site hosting moves off mail-tba to Cloudflare,
  - non-mail infra/apps move to dev-2026,
  - mail-tba VM is repurposed as DR standby for critical data (Hub/auth/users/invoicing data continuity).
- Mail provider target:
  - **Resend** for app/transactional outbound email,
  - **Proton Mail** for human mailbox/ops email lanes.

## Current baseline snapshot (2026-04-25)
- Mac PM2: 14 online
- dev-2026 Docker: mixed multi-stack runtime, with restart loops in vault + plausible events DB
- dev-2026 PM2: only pm2-logrotate

### DNS/Web pre-cutover check (17:43 CET)
- `kraliki.com`, `www.kraliki.com`, `verduona.com`, `www.verduona.com`, `crm.kraliki.com` → `91.99.176.2` (mail-tba lane)
- `identity.verduona.dev`, `hub.verduona.dev`, `agentjack-dev.verduona.dev` → `138.201.54.142` (dev-2026 lane)
- HTTPS probes returned expected live responses (200/3xx) on all above hosts.

## Next 3 actions
1. Build a lane map: every service tagged as `cloudflare-websites`, `dev-2026-runtime`, `councilnow-showcase`, or `dr-standby`.
2. Standardize dev-2026 deploy runbooks to Ubuntu-container flow using CouncilNow as the template.
3. Execute first migration slice: websites off mail-tba to Cloudflare, then convert mail-tba into DR standby (backup/replica/restore target) for critical data.

## Tasklist additions (founder request)
- Build infra handling to pro level with agent-first control paths:
  - Cloudflare via MCP/API automation lane (DNS, Pages, zones, verification checks).
  - Namecheap via MCP/API automation lane (DNS + nameserver operations).
  - Rule: choose the easiest reliable control path for agent execution (MCP first, direct API fallback).

## New artifact
- `/Users/matejhavlin/github/management/DR-STANDBY-PLAN-MAIL-TBA.md`
