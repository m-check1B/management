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
