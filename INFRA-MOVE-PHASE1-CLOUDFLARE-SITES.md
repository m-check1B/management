# Infra Move — Phase 1 (Sites to Cloudflare)

_Last updated: 2026-04-25 18:02 CET_

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
- Sites above currently resolve to `91.99.176.2`
- HTTPS currently live (200/3xx)

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
- Restore previous DNS for affected site hostnames to `91.99.176.2`
- Restart mail-tba static containers if needed
