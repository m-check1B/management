# Status — All Projects & Infrastructure

_Last updated: 2026-04-26 08:39 CEST_

## Products

| Project | Location | Status | Priority | Notes |
|---------|----------|--------|----------|-------|
| **AgentJack** | `/github/agentjack` | 🔄 Active hardening | P1 — THE product | Dedicated gateway ownership fixed (`agentjack-gateway.service` active, Hermes gateway disabled). Telegram lane is healthy. Private webchat local readyz path still blocked. |
| **CouncilNow** | `/github/applications/advisory-council-app` + `/github/websites/councilnow.com` | 🔄 Active | P1 — Revenue first | Billing/auth lane previously verified repeatedly; focus remains customer acquisition and conversion. |
| **open-kraliki** | `/github/open-kraliki` | ✅ Stable substrate | P2 | Automation templates backing AgentJack Automations surface. |

## Workbench / Internal

| Tool | Status | Notes |
|------|--------|-------|
| **OpenClaw** | ⚠️ Running with critical security findings | Operational control plane, but security audit currently shows critical posture gaps (gateway auth + small-model policy). |
| **GStack** | ✅ Installed | Primary execution cookbook. |
| **GBrain** | ✅ Installed | Knowledge infra active (Postgres + vector lane). |

## Shared Infrastructure

| Component | Status | Notes |
|-----------|--------|-------|
| **Zitadel** | ✅ Active | identity.verduona.dev |
| **Hub** | ✅ Active | Shared services + billing path |
| **dev-2026** | ✅ Online | Main execution host |
| **Cloudflare / Namecheap / Hetzner** | ✅ Active | Core infra providers operational |

## Current Audit Snapshot (OpenClaw vs AgentJack)

Source file: `OPENCLAW-vs-AGENTJACK-AUDIT-2026-04-26.md`

- OpenClaw: mature ops lane, but largest immediate risk is security posture.
- AgentJack: runtime isolation and Telegram polling conflict resolved; main gaps are private web surface reliability + integration maturity.

## Known Issues (current)

1. **OpenClaw security criticals unresolved**
   - Gateway auth exposure and small-model sandbox/tool policy findings remain open.
2. **AgentJack private webchat health blocked**
   - Local/private readyz path (`127.0.0.1:4173`) not healthy while public AG WebUI readyz is green.
3. **AgentJack Telegram continuity proof still human-token gated**
   - Runtime transport is healthy; doctor can remain in attention until proof reply is received.
4. **AgentJack integrations extraction gap**
   - Current overview: healthy=2, attention=4, planned=13.
5. **OpenClaw state-integrity hygiene warnings**
   - Multi-state-dir / orphan transcript warnings still reported by doctor.

## Recently Resolved

- ✅ **Telegram 409 polling conflict root cause resolved** by removing competing gateway ownership and running dedicated AgentJack gateway service.
