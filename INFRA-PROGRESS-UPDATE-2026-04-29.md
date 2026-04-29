# INFRA PROGRESS UPDATE — 2026-04-29 11:15 CEST

## Completed Today

### 1. kraliki.com Bare Domain → Cloudflare Pages ✅
- Deleted old A records for `kraliki.com` and `www.kraliki.com` (→ 91.99.176.2 mail-tba)
- Created CNAME records → `kraliki-marketing.pages.dev` (proxied)
- Clicked "Reactivate" + "Check DNS records" on CF Pages dashboard
- Both domains now **Active** on kraliki-marketing Pages project
- **HTTP 200 confirmed** for both `kraliki.com` and `www.kraliki.com`

### 2. Cloudflare Email Routing — ALL 3 DOMAINS ✅
- **kraliki.com**: Catch-all → `matej.havlin@protonmail.com` (enabled)
- **verduona.com**: Catch-all → `matej.havlin@protonmail.com` (enabled)
- **perfectmission.co.uk**: Catch-all → `matej.havlin@protonmail.com` (enabled)
- Destination address verified (Matej clicked the CF verification link in ProtonMail)
- Key API discovery: catch-all rules use `/api/v4/zones/{zoneId}/email/routing/rules/catch_all` endpoint (PUT), not the generic `/rules/{id}` endpoint

### 3. accounts.kraliki.com → Zitadel Auth ✅
- Zitadel's `ExternalDomain=identity.verduona.dev` rejects unknown Host headers
- `customRequestHeaders Host` rewrite doesn't work in Traefik (Host is special, `passHostHeader` overrides)
- Solution: Traefik `redirectRegex` middleware — `accounts.kraliki.com/*` → `https://identity.verduona.dev/*` (307)
- Path preservation confirmed: `/ui/console` → `/ui/console`, `/.well-known/openid-configuration` → same
- OIDC issuer confirmed: `https://identity.verduona.dev`

### 4. Traefik Config Fixes ✅
- Fixed `passHostHeader` (accidentally changed all services to false, restored correct values)
- Removed `security-headers@file` from accounts-kraliki router (interfered with redirect)
- `production-vms.yml` stable with all 7 routes + accounts redirect middleware

## Architecture State

### DNS (kraliki.com zone, ~33 records)
| Record | Type | Target | Status |
|--------|------|--------|--------|
| kraliki.com | CNAME | kraliki-marketing.pages.dev | Active ✅ |
| www.kraliki.com | CNAME | kraliki-marketing.pages.dev | Active ✅ |
| accounts.kraliki.com | A | 138.201.54.142 | → redirect to identity.verduona.dev |
| api.kraliki.com | A | 138.201.54.142 | → Hub API |
| hub.kraliki.com | A | 138.201.54.142 | → Hub |
| agentjack.kraliki.com | CNAME | agentjack-marketing.pages.dev | Active ✅ |

### Email Routing
| Domain | Catch-all | Destination | Status |
|--------|-----------|-------------|--------|
| kraliki.com | ✅ | matej.havlin@protonmail.com | Enabled |
| verduona.com | ✅ | matej.havlin@protonmail.com | Enabled |
| perfectmission.co.uk | ✅ | matej.havlin@protonmail.com | Enabled |

### Auth
| URL | Behavior | Status |
|-----|----------|--------|
| identity.verduona.dev | Zitadel OIDC issuer | ✅ Working |
| accounts.kraliki.com | 307 → identity.verduona.dev | ✅ Working |

## Remaining Tasks
- ~~Hub billing DB config~~ — ALREADY WORKING. Hub `/api/billing/status` returns `active` with 4 tiers (free/starter/pro/enterprise). Stripe keys configured.
- Fly.io old services scale-down
- mail-tba rename to `kraliki-dr-standby` + Ubuntu 26.04 rebuild
- Resend domain verification for additional domains (currently only councilnow.com)
- ProtonMail send-as configuration (Matej's manual task)
- AgentJack worker dispatch layer development

## 14:40 CEST — YOLO Infrastructure Audit

### All Production Services: GREEN 🟢
- All 3 VMs RUNNING (Ubuntu 26.04 LTS)
- Zitadel ✅, Hub ✅ (billing active), AgentJack ✅, Runner Pool ✅
- Runner→Control cross-VM connectivity ✅
- Traefik Up 3+ hours, all routes stable
- CouncilNow, kraliki.com, agentjack.kraliki.com: all HTTP 200

### Backup Health: HEALTHY 🟢
- PG tier0: 2026-04-29 14:30 ✅
- PG tier1: 2026-04-29 14:00 ✅
- AgentJack state: 2026-04-29 14:30 ✅
- GitHub → Hetzner: fresh ✅

### Security Findings (flagged for review) 🟡
1. **All 3 VMs have ufw firewall INACTIVE** — no firewall rules. Traefik forwards traffic, but VMs accept from any source on internal network.
2. **Advisory Zitadel dev (port 8086) publicly accessible** — returns 302 to login page. Should be firewalled or container stopped.
3. **Kiki API (port 8300) publicly accessible** — should be behind Traefik only.
4. **RDP (port 3389) publicly accessible** — xrdp running on host, should be firewalled.

### Telegram Bot Status 🟡
- Bot `@AxisMacBot` confirmed working via API
- No paired chat exists — dmPolicy="pairing" requires Matej to message the bot first
- Created 30-min auto-report cron job (fires once pairing established)
- Status report written: `management/telegram-status-report.md`

### Still Blocked
- **Telegram**: needs Matej to message @AxisMacBot
- **Fly.io**: token expired (2026-02-16), needs re-auth
- **mail-tba → kraliki-dr-standby**: plan exists, execution pending
- **VM firewalls (UFW)**: needs careful implementation to avoid breaking Traefik routing
- **ProtonMail send-as**: Matej configures in ProtonMail web UI
