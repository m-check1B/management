# Cloudflare Pages Marketing Deploy — 2026-04-26

_Status: Pages deployments live; custom-domain cutover blocked by missing Cloudflare DNS permission._

## Built

- `websites/councilnow.com` → `pnpm build` ✅
- `websites/kraliki.com` → `pnpm build` ✅
- `websites/verduona.com` → `npm run build` ✅
- `websites/perfectmission.co.uk` → `npm run build` ✅
- `github/blog` → `npm run build` ✅
- `websites/agentjack.kraliki.com` → `npm run build` ✅

## Deployed to Cloudflare Pages

| Surface | Pages project | URL | Verify |
|---|---|---|---|
| CouncilNow marketing | `councilnow-marketing` | https://councilnow-marketing.pages.dev/ | HTTP 200, headline present |
| Kraliki marketing | `kraliki-marketing` | https://kraliki-marketing.pages.dev/ | HTTP 200, Kraliki marker present |
| Verduona marketing | `verduona-main` | https://verduona-main.pages.dev/ | HTTP 200, Verduona marker present |
| PerfectMission marketing | `perfectmission-marketing` | https://perfectmission-marketing.pages.dev/ | HTTP 200, Perfect marker present |
| m-check1B blog | `m-check1b` | https://m-check1b.pages.dev/ | HTTP 200, AI Systems Consultant marker present |
| AgentJack marketing | `agentjack-marketing` | https://agentjack-marketing.pages.dev/ | HTTP 200, AgentJack marker present |

## Custom Domain State

Cloudflare Pages custom domains were attached where possible, but they are not serving from Pages yet:

| Project | Custom domains | Status |
|---|---|---|
| `councilnow-marketing` | `councilnow.com`, `www.councilnow.com` | `pending` |
| `kraliki-marketing` | `kraliki.com`, `www.kraliki.com` | `deactivated` |
| `verduona-main` | `verduona.com`, `www.verduona.com` | `deactivated` |
| `perfectmission-marketing` | `perfectmission.co.uk`, `www.perfectmission.co.uk` | `pending` |
| `m-check1b` | `m-check1b.com`, `www.m-check1b.com` | `pending` |
| `agentjack-marketing` | `agentjack.kraliki.com` | `pending` |

Available Cloudflare tokens can deploy Pages and attach Pages domains, but DNS record read/edit calls return `403 Forbidden`. Final production cutover needs a Cloudflare token with Zone DNS edit permission or manual DNS changes in the dashboard.

## Current production public-domain proof

- `councilnow.com` still serves via dev-2026/Traefik and remains healthy for the app path.
- `kraliki.com`, `verduona.com`, and `perfectmission.co.uk` still return Apache headers from the VPS deploy path, not Pages.
- Pages defaults are live and ready for domain cutover.

## Advisory Council app proof

- `https://councilnow.com/health` returned `{"status":"ok","version":"0.4.0","database":"ok"...}`.
- `https://councilnow.com/app` returned HTTP 200.
- dev-2026 LXC `advisory-council` containers are running the final deploy tag `councilnow-sidecar-preserve-20260426211257` for API/hub/frontend, with API and hub healthy.

## DNS cutover note

CouncilNow needs a split before apex cutover: marketing on Pages and app/API/auth/health/login/shared routes on dev-2026. Existing infra docs already call out `app.councilnow.com` as the clean split target, but DNS is not configured yet. Do not flip `councilnow.com` apex fully to Pages without preserving the app routes.
