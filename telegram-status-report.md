# Telegram Status Report — 2026-04-29 14:40 CEST

## 🟢 Production Infrastructure: ALL HEALTHY

### VMs (Ubuntu 26.04 LTS)
| VM | IP | Status |
|---|---|---|
| prod-control-plane | 10.181.203.8 | RUNNING |
| agentjack-control | 10.181.203.208 | RUNNING |
| agentjack-runner-pool | 10.181.203.43 | RUNNING |

### Services
| Service | Endpoint | Status |
|---|---|---|
| Zitadel (Auth) | identity.verduona.dev | ✅ ok |
| Hub (API/Billing) | hub.verduona.dev | ✅ ok (billing active, 4 tiers) |
| AgentJack Gateway | agentjack-dev.verduona.dev | ✅ ok |
| AgentJack Runner Pool | 10.181.203.43:8801 | ✅ ok |
| Runner→Control connectivity | cross-VM | ✅ ok |
| Traefik | dev-2026 | ✅ Up 3+ hours |
| CouncilNow | councilnow.com | ✅ HTTP 200 |
| kraliki.com | kraliki.com | ✅ HTTP 200 |
| agentjack.kraliki.com | agentjack.kraliki.com | ✅ HTTP 200 |

### DNS (kraliki.com zone — ~33 records)
- All `*.kraliki.com` A records → 138.201.54.142 (proxied via Cloudflare)
- `kraliki.com` + `www.kraliki.com` → Cloudflare Pages (kraliki-marketing.pages.dev)
- `agentjack.kraliki.com` → Cloudflare Pages (agentjack-marketing.pages.dev)
- `accounts.kraliki.com` → 307 redirect to identity.verduona.dev

### Email Routing (all 3 domains → ProtonMail)
| Domain | Status | Catch-all |
|---|---|---|
| kraliki.com | Enabled, DNS locked | → matej.havlin@protonmail.com ✅ |
| verduona.com | Enabled, DNS locked | → matej.havlin@protonmail.com ✅ |
| perfectmission.co.uk | Enabled, DNS locked | → matej.havlin@protonmail.com ✅ |

Destination address verified ✅

### Backups
| Type | Status |
|---|---|
| PG tier0 (latest) | 2026-04-29 14:30 ✅ |
| PG tier1 (latest) | 2026-04-29 14:00 ✅ |
| AgentJack state | 2026-04-29 14:30 ✅ |
| GitHub → Hetzner | fresh (today) ✅ |
| OpenClaw archive | 33h old ✅ |
| Overall health | HEALTHY |

### Hub Billing (CONFIRMED WORKING)
- `/api/billing/status` → active, 4 tiers (Free/Starter/Pro/Enterprise)
- Stripe live keys configured
- No database flag needed — Stripe key presence is sufficient

## 🟡 Pending Items

### Telegram Bot (@AxisMacBot)
- Bot is configured but **no chat pairing exists**
- **Action needed:** Matej messages @AxisMacBot on Telegram to establish pairing
- Once paired: auto status reports will deliver every 30 min

### Fly.io Old Services
- Token expired (last login 2026-02-16)
- Old DNS records already deleted from kraliki.com zone
- **Blocked:** needs re-auth (`flyctl auth login`)

### ProtonMail Send-as
- Matej needs to configure "Send as" in ProtonMail for:
  - matej@verduona.com
  - matej@kraliki.com  
  - matej@perfectmission.co.uk
- Cloudflare catch-all will deliver ProtonMail's verification emails

### DR: mail-tba → kraliki-dr-standby
- mail-tba still on Ubuntu 22.04 (3.7GB RAM, 75GB disk 38% used)
- Plan exists: rename + rebuild on 26.04 LTS
- Not urgent — active-passive DR standby

## 📋 What Was Done Today

1. ✅ Rebuilt all 3 production VMs on Ubuntu 26.04 LTS
2. ✅ Full production migration — all traffic on new VMs
3. ✅ All kraliki.com DNS records created + dead records cleaned
4. ✅ accounts.kraliki.com → Zitadel redirect (redirectRegex)
5. ✅ Email routing on all 3 domains → ProtonMail
6. ✅ Hub billing confirmed working (Stripe live keys, 4 tiers)
7. ✅ Website audit — all domains verified
8. ✅ Auth flows verified (Google SSO, Zitadel OIDC)
9. ✅ Cloudflare Email Routing destination verified
10. ✅ kraliki.com bare domain → Cloudflare Pages
11. ✅ Traefik config fixes (dynamic.yml, production-vms.yml)
