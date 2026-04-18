# Status — All Projects & Infrastructure

_Last updated: 2026-04-18 23:21_

## Products

| Project | Location | Status | Priority | Notes |
|---------|----------|--------|----------|-------|
| **AgentJack** | `/github/agentjack` | 🔄 Development | P1 — THE product | Agentic orchestration. Agent core first → apps → surpass Garry |
| **CouncilNow** | `/github/applications/advisory-council-app` + `/github/websites/councilnow.com` | 🔄 Active | P1 — Revenue first | Multi-agent debate. Run as real business ASAP |
| **open-kraliki** | `/github/open-kraliki` | ✅ Stable | P2 | Automation templates. Substrate for AgentJack Automations |

## Marketing

| Site | Status | Notes |
|------|--------|-------|
| **kraliki.com** | Live | Brand presence |
| **verduona.com** | Live | Brand presence |
| **councilnow.com** | Live | Product site for CouncilNow |

## Shared Infrastructure

| Component | Status | Notes |
|-----------|--------|-------|
| **Zitadel** | Active | identity.verduona.dev — SSO/auth for all products |
| **Hub** | Active | Shared services (billing, etc.) |

## Heritage (Not Active Dev)

| Project | Status | Purpose |
|---------|--------|---------|
| **kraliki-monorepo** | ⛔ Stopped | Mine for ideas/solutions/code. Don't build on it. |

## Internal Tooling

| Tool | Status | Notes |
|------|--------|-------|
| **GStack** | Installed | Primary cookbook. 23 specialist commands. Watch for updates. |
| **GBrain** | Installed | Knowledge infra. Local Postgres + pgvector. Watch for updates. |
| **OpenClaw** | Running | Gateway/orchestration. Axis lives here. |

## Infrastructure Health

| Component | Location | Status | Last Check |
|-----------|----------|--------|------------|
| **Mac (primary)** | Local | ✅ Online | 2026-04-18 |
| **dev-2026** | Hetzner | ❓ Unknown | Needs check |
| **Mail VM** | Hetzner | ❓ Unknown | Needs check |
| **Cloudflare** | Cloud | ✅ Active | — |
| **Namecheap** | Cloud | ✅ Active | — |
| **Hetzner** | Cloud | ✅ Active | — |
| **PM2 (local)** | Mac | ⚠️ 16/16 online but was empty Apr 14-18 | 2026-04-18 |
| **Docker** | Mac + dev-2026 | ✅ Containers healthy | 2026-04-18 |
| **Embedding server** | Mac :1934 | ✅ Running | 2026-04-18 |

## Known Issues

1. **Telegram bot 409 conflict** — duplicate bot polling same token (7.8K occurrences)
2. **PM2 empty on restart** — needs `pm2 resurrect` each time
3. **AC Hub `kraliki_common` missing** — AUVS checks fail
4. **Autodev stuck loops** — councilnow + kraliki-monorepo skipping AI cycles
5. **Exec preflight block** — Python scripts with flags refused by OpenClaw
