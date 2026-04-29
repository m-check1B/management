# Website Audit Proof — 2026-04-29 10:35 CEST

## DNS Changes Completed
- Deleted `agentjack` A record (→138.201.54.142) from kraliki.com zone
- Deleted `swarm` CNAME (→kraliki-swarm.fly.dev) from kraliki.com zone
- Added `agentjack` CNAME → `agentjack-marketing.pages.dev` (proxied) via CF API
- Zone ID: `7f61e36ba67266b185e5f52696fc460c`
- Record ID: `493faee3f48b18a1c4e7c9a353b1287f`

## Full Audit Results

### kraliki.com Subdomains
| Domain | Status | Notes |
|--------|--------|-------|
| accounts.kraliki.com | 302 | Redirects to Zitadel login. ⚠️ Login page returns 404 (ExternalDomain issue) |
| api.kraliki.com | 405 (HEAD) / 200 (GET /health) | Hub health OK, env=production-internal |
| hub.kraliki.com | 405 (HEAD) / 200 (GET /health) | Hub health OK, env=production-internal |
| agentjack.kraliki.com | 200 | ✅ Cloudflare Pages marketing site serving correctly |

### verduona.dev Subdomains
| Domain | Status | Notes |
|--------|--------|-------|
| identity.verduona.dev | 302→200 | Zitadel OIDC, issuer verified |
| hub.verduona.dev | 308→200 | Hub health OK, env=staging |
| agentjack-dev.verduona.dev | 404 (root) / 200 (/health) | AgentJack health OK |

### Marketing/Root Sites
| Domain | Status | Notes |
|--------|--------|-------|
| kraliki.com | 200 | Cloudflare Pages (kraliki-marketing) |
| www.kraliki.com | 308→kraliki.com | Redirect OK |
| councilnow.com | 200 | Cloudflare Pages (councilnow-marketing) |
| www.councilnow.com | 200 | OK |
| verduona.com | 200 | Cloudflare Pages (verduona-main) |
| www.verduona.com | 308→verduona.com | Redirect OK |
| perfectmission.co.uk | 200 | Cloudflare Pages (perfectmission-marketing) |
| www.perfectmission.co.uk | 308→perfectmission.co.uk | Redirect OK |

### Cloudflare Pages Dev Domains
| Domain | Status |
|--------|--------|
| m-check1b.pages.dev | 200 |
| councilnow-marketing.pages.dev | 200 |
| agentjack-marketing.pages.dev | 200 |
| perfectmission-marketing.pages.dev | 200 |
| verduona-main.pages.dev | 200 |
| kraliki-marketing.pages.dev | 200 |

### Auth Flows
| Endpoint | Status | Notes |
|----------|--------|-------|
| identity.verduona.dev OIDC | ✅ Working | issuer: https://identity.verduona.dev |
| accounts.kraliki.com → Zitadel | ⚠️ 302→404 | ExternalDomain mismatch (Zitadel config issue) |

### Health Endpoints
| Endpoint | Status | Details |
|----------|--------|---------|
| api.kraliki.com/health | ✅ ok | kraliki-hub, production-internal |
| hub.kraliki.com/health | ✅ ok | kraliki-hub, production-internal |
| hub.verduona.dev/health | ✅ ok | kraliki-hub, staging |
| agentjack-dev.verduona.dev/health | ✅ ok | agentjack |

## Known Issues
1. **accounts.kraliki.com Zitadel 404**: Zitadel's ExternalDomain=identity.verduona.dev doesn't accept accounts.kraliki.com as a hostname. Needs config change or Traefik host rewrite.
2. **kraliki.com bare domain**: Points to mail-tba (91.99.176.2) but Cloudflare Pages serves content. The Pages project (kraliki-marketing) handles it.
3. **agentjack.kraliki.com DNS propagation**: New CNAME record may take a few minutes to propagate globally. Direct Cloudflare IP test confirms it works (HTTP 200).

## Auth Flow Testing (Pending)
- Google SSO button on CouncilNow/AgentJack: needs browser testing
- Zitadel login method: needs browser testing on identity.verduona.dev
