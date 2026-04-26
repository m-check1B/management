# Kiki Cloudflare Parity — 2026-04-26

_Status: Parity green; apex DNS cutover still intentionally not executed_

## Result

Kiki API parity blocker is cleared.

- Stable origin: `https://kiki-api.verduona.dev`
- Origin host: dev-2026 (`138.201.54.142`)
- Cloudflare Pages proxy deployed:
  - `https://kraliki-marketing.pages.dev/api/kiki/*`
  - `https://verduona-main.pages.dev/api/kiki/*`
- Both Pages targets now return JSON instead of Svelte HTML fallback.

## 1. Origin: kiki-api.verduona.dev on dev-2026

### Done
- Verified `kiki-api` container running on dev-2026: healthy, port `8300`.
- Connected `kiki-api` to Traefik network `compose_default`.
- Created Traefik dynamic route on dev-2026:
  - `/home/adminmatej/github/infra/compose/traefik-dynamic/kiki-api.yml`
  - Router: `kiki-api.verduona.dev` → service `kiki-api` → `http://172.24.0.5:8300`
  - TLS via Let's Encrypt.
- Added explicit Cloudflare DNS record:
  - Zone: `verduona.dev`
  - `A kiki-api.verduona.dev -> 138.201.54.142`
  - Proxied: false

### Verified

```bash
dig +short A kiki-api.verduona.dev @1.1.1.1
# 138.201.54.142

curl -si https://kiki-api.verduona.dev/health
# HTTP/2 200
# content-type: application/json
```

## 2. Cloudflare Pages functions

Created and deployed `functions/api/kiki/[[path]].js` in both website projects:

- `/Users/matejhavlin/github/websites/kraliki.com/functions/api/kiki/[[path]].js`
- `/Users/matejhavlin/github/websites/verduona.com/functions/api/kiki/[[path]].js`
- Also prepared in `/Users/matejhavlin/github/websites-new/...` during the infra lane.

Function behavior: strips `/api/kiki` prefix and proxies to `https://kiki-api.verduona.dev/<path>`.

### Commits

- `/Users/matejhavlin/github/websites`: `a48b68e Add Kiki API Pages proxy functions`
- `/Users/matejhavlin/github/websites-new`: `ee53ff8 Add Kiki API Pages proxy functions`

## 3. Cloudflare Pages deployments

Deployed via Wrangler using the Cloudflare Pages token from `/Users/matejhavlin/github/secrets/_archive/cloudflare-pages.env`.

- Kraliki deployment: `https://bfb6ae68.kraliki-marketing.pages.dev`
- Verduona deployment: `https://396421f3.verduona-main.pages.dev`

## 4. Parity verification

```bash
curl -si https://kraliki-marketing.pages.dev/api/kiki/health
# HTTP/2 200
# content-type: application/json

curl -si https://verduona-main.pages.dev/api/kiki/health
# HTTP/2 200
# content-type: application/json

curl -si https://bfb6ae68.kraliki-marketing.pages.dev/api/kiki/health
# HTTP/2 200
# content-type: application/json

curl -si https://396421f3.verduona-main.pages.dev/api/kiki/health
# HTTP/2 200
# content-type: application/json
```

## 5. Remaining decision

Apex website DNS cutover for `kraliki.com` / `verduona.com` was **not** executed in this pass. The blocker it depended on is now cleared, but the cutover itself is a separate production DNS action.

Recommended next step: cut over apex/www DNS to Cloudflare Pages in a controlled window, then verify:

- root page loads
- `/en/` page loads
- `/api/kiki/health` returns JSON
- CouncilNow cross-links still render
