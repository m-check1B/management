# Infra Move — Phase 1 (Sites to Cloudflare)

_Last updated: 2026-04-26 00:22 CET_

## Source of truth
- `/Users/matejhavlin/github/infra/MAIL-VM-MIGRATION-PLAN.md`
- `/Users/matejhavlin/github/infra/PRODUCTION-ARCHITECTURE.md`

## Goal
Move static websites from mail-tba (`91.99.176.2`) to Cloudflare Pages/Websites without touching mail DNS in this phase.

Note: end-state target is repurposing `mail-tba` as DR standby for critical data after all migration phases.

## Scope in this phase
- `kraliki.com`
- `www.kraliki.com`
- `verduona.com`
- `www.verduona.com`

## Do NOT touch in this phase
- `mail.kraliki.com`
- MX records
- SPF / DKIM / DMARC

Mail target reminder (later phase):
- **Resend** for app/transactional outbound email
- **Proton Mail** for mailbox/ops email

## Pre-cutover baseline (captured)
- Sites above currently resolve to `91.99.176.2` (re-verified 2026-04-25 22:35 CET)
- HTTPS currently live (re-verified):
  - `https://kraliki.com` → `200`
  - `https://www.kraliki.com` → `308` to apex
  - `https://verduona.com` → `200`
  - `https://www.verduona.com` → `308` to apex
- Mail guardrail baseline:
  - `kraliki.com MX` → `10 mail.kraliki.com.`
  - SPF/DMARC present and unchanged on both domains

## Exact Phase 1 DNS change-set (prepared)

| Host | Current record | Target record (Phase 1) | Change allowed now? |
|---|---|---|---|
| `kraliki.com` | `A 91.99.176.2` | Cloudflare Pages (`kraliki-marketing.pages.dev`) | Yes |
| `www.kraliki.com` | `A 91.99.176.2` | CNAME/redirect to Cloudflare Pages + apex (`kraliki-marketing.pages.dev`) | Yes |
| `verduona.com` | `A 91.99.176.2` | Cloudflare Pages (`verduona-main.pages.dev`) | Yes |
| `www.verduona.com` | `A 91.99.176.2` | CNAME/redirect to Cloudflare Pages + apex (`verduona-main.pages.dev`) | Yes |
| `mail.kraliki.com` | `A 91.99.176.2` | **NO CHANGE** | **No** |
| MX/SPF/DKIM/DMARC | live mail records | **NO CHANGE** | **No** |

### Cloudflare Pages target bindings (verified)
- `kraliki-marketing.pages.dev` and `verduona-main.pages.dev` are active Cloudflare Pages projects in account `21065b2afe7f6e807a2b9bd560ea6c38`.
- Production branch on both projects is `beta`.
- Fresh beta deployments were pushed from local source on 2026-04-25, and title parity now matches production for both apex sites.

### Blockers to execute cutover
- Functional parity gap: `/api/kiki/*` currently works on live domains via server-side route, but Pages targets return HTML app fallback at those paths (HTTP 200 with page content, not API JSON). DNS cutover before fixing this would break Kiki API calls on both domains.

### Founder decision (2026-04-26 00:22 CET)
- Websites are static and should stay static on Cloudflare Pages.
- Kiki is a backend, not part of the static website artifact.
- Move/serve the Kiki backend from a **dev-2026 container** and treat it as the stable API origin.
- Cloudflare Pages should only provide `/api/kiki/*` same-origin proxy behavior to the dev-2026 backend.

### Unblock plan (next)
1. Stand up or verify the Kiki backend container on `dev-2026` (expected internal service equivalent to current `http://host.docker.internal:8300`).
2. Expose it through a stable dev-2026 origin that survives apex cutover (preferred: a dedicated hostname such as `kiki-api.verduona.dev` or equivalent explicit Traefik route to `138.201.54.142`).
3. Add explicit `/api/kiki/*` proxy behavior to both Pages projects (Cloudflare Function or equivalent edge rewrite) pointing at that stable dev-2026 origin.
4. Re-run parity checks:
   - `curl -si https://kraliki-marketing.pages.dev/api/kiki/health`
   - `curl -si https://verduona-main.pages.dev/api/kiki/health`
   - expect `content-type: application/json` and healthy payload from the dev-2026 backend.
5. Execute DNS cutover only after parity passes.

## Execution checklist
1. Build and publish static artifacts to Cloudflare Pages projects.
2. Attach custom domains for apex + www.
3. Verify Cloudflare origin/site health before DNS cut.
4. Cut DNS for the four site hostnames only.
5. Validate from public internet:
   - `curl -I https://kraliki.com`
   - `curl -I https://www.kraliki.com`
   - `curl -I https://verduona.com`
   - `curl -I https://www.verduona.com`
6. Browser smoke-check both sites.
7. Keep mail-tba static containers running until validation is green.
8. Retire mail-tba static containers only after green verification.

## Rollback
- Restore previous DNS for affected site hostnames to `91.99.176.2`:
  - `kraliki.com`
  - `www.kraliki.com`
  - `verduona.com`
  - `www.verduona.com`
- Verify mail DNS remains untouched (`mail.kraliki.com`, MX/SPF/DKIM/DMARC).
- Restart mail-tba static containers if needed.
- Re-run public checks:
  - `curl -I https://kraliki.com`
  - `curl -I https://www.kraliki.com`
  - `curl -I https://verduona.com`
  - `curl -I https://www.verduona.com`
