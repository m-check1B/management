# Status — All Projects & Infrastructure

_Last updated: 2026-04-28 17:12 CEST_

## Current operating mode

**Paused by Matej.** No proactive work or scheduled execution until Matej gives an explicit start/resume command.

- OpenClaw cron jobs: **0 enabled** after pause cleanup.
- Heartbeat: pause override active; should no-op and return `HEARTBEAT_OK` while `STOP_FOR_TODAY.md` exists.
- Active subagents: none at pause check.
- Running background commands: none at pause check.

## Products

| Project | Location | Status | Priority | Notes |
|---------|----------|--------|----------|-------|
| **AgentJack** | `/github/agentjack` | ✅ Clean / gated locally | P1 — THE product | Yolo cleanup completed and pushed. Current HEAD `ca0a98a`; repo clean. Full Python suite passed (`582 passed, 6 skipped, 16 subtests passed`), AG WebUI check/strict JS/build passed, and product vertical check/build/smoke gates passed. No live/public deploy after this cleanup. |
| **CouncilNow** | `/github/applications/advisory-council-app` + `/github/websites/councilnow.com` | ✅ Revenue path verified | P1 — Revenue first | Latest repeated auth→checkout proof remained green on 2026-04-28. Continue using precise language: billing/revenue path is verified; product-quality readiness is a separate claim. |
| **open-kraliki** | `/github/open-kraliki` | ✅ Stable substrate | P2 | Automation templates backing AgentJack Automations surface. |

## Workbench / Internal

| Tool | Status | Notes |
|------|--------|-------|
| **OpenClaw** | ✅ Operational / paused | Local health was green at last heartbeat: PM2 14 online, Hub ok, OpenViking ok, embeddings reachable. Proactive cron/heartbeat execution is paused by founder instruction. |
| **Hermes** | ⚠️ Legacy/reference lane | Not part of the active local baseline unless Matej explicitly reintroduces it. Keep token/channel ownership separate from OpenClaw/AgentJack to avoid polling conflicts. |
| **GStack** | ✅ Installed | Primary execution cookbook. |
| **GBrain** | ✅ Installed | Knowledge infra active (Postgres + vector lane). |

## Shared Infrastructure

| Component | Status | Notes |
|-----------|--------|-------|
| **Zitadel** | ✅ Active | identity.verduona.dev |
| **Hub** | ✅ Active | Shared services + billing path |
| **dev-2026** | ✅ Online | Main execution host |
| **Cloudflare / Namecheap / Hetzner** | ✅ Active | Core infra providers operational; no DNS/billing/public deployment changes without explicit approval. |

## AgentJack 2026-04-28 progress snapshot

Pushed cleanup/fix commits:

- `898c669` — stabilize beta smoke gates
- `43ced52` — include app section canvas data
- `677087e` — align app-boundary assertions with section loader
- `f6f99d0` — wait for chat composer UI updates
- `95f34dd` — integrate product vertical contracts
- `93c8fec` — bound AG WebUI bridge CLI calls
- `400ae29` — record ops continuity updates
- `ca0a98a` — sync SvelteKit before builds

Key fixes:

- AG WebUI beta cleanup and launcher/boundary stabilization.
- Shared chat page sanitization path.
- Garage smoke env-auth setup.
- Notes frontend current branding expectation.
- Webchat E2E process isolation/dynamic port behavior.
- Collective/Comms SvelteKit config/bootstrap repair.
- Comms form label/accessibility fixes.
- Product vertical contract app check/build/smoke coverage.

## Known current caveats

1. **No live AgentJack deploy after yolo cleanup**
   - Local/main is green and pushed; public/live deployment still requires explicit deploy instruction and deploy verification.
2. **Cron set needs rebuild, not blanket restore**
   - 39 stored jobs, 0 enabled. On resume, re-enable a lean set only.
3. **Autonomous VM agent lab is on hold**
   - Matej explicitly said not to do the OpenClaw/Hermes/AgentJack VM-with-agents plan yet. Do not create VM agents, provision tokens, or start self-development loops.
4. **OpenClaw security posture still needs a fresh audit before claims**
   - Do not reuse old “critical” or “green” claims without a current doctor/security pass.
5. **Hermes/OpenClaw/AgentJack token ownership must stay separated**
   - Prior Telegram/token drift caused conflicts; if the VM lab is ever approved later, it must use scoped staging tokens only.

## Recently resolved

- ✅ AgentJack repo churn resolved to clean `main` at `ca0a98a`.
- ✅ AgentJack local product/test gates are green.
- ✅ Cron/heartbeat noise paused after Matej’s stop command.
- ✅ Management docs updated to reflect the new state.
