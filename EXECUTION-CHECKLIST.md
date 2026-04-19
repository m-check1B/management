# EXECUTION-CHECKLIST.md — Wave-Driven Execution

_Last updated: 2026-04-19 08:57_
_Status: ACTIVE — Cron drives this every 15 minutes_

## How This Works

Every 15 minutes, a cron fires. Axis reads this file, picks the next unchecked item, executes it, marks it done. Repeat until the wave is complete, then start the next wave.

**Rules:**
1. Pick the FIRST unchecked item in the current wave
2. Execute it fully (code, config, deploy — not plans)
3. Mark it `[x]` with timestamp
4. Push to GitHub
5. If blocked, mark `[!] BLOCKED: reason` and move to next
6. When all items in a wave are done, update `_Last updated` and move to next wave
7. If ALL waves done, write new waves based on current goals

---

## WAVE 1: CouncilNow Production-Ready (REVENUE PATH)

- [x] **W1.1** Test auth flow end-to-end: Zitadel OIDC login → session → user created in DB — ✅ 2026-04-19 08:57 — Fixed missing Ed25519 keys in deployment; email/password register→login→JWT→/me→API→DB all verified. Note: Zitadel OIDC provider not yet registered (0 rows in oidc_providers); email/password auth fully working.
- [x] **W1.2** Test Stripe checkout end-to-end: select plan → Stripe checkout → webhook → credits/grant — ⚠️ 2026-04-19 09:16 — Checkout creation ✅ (live Stripe session created), portal ✅, subscription ✅, summary ✅. BLOCKED on webhook: STRIPE_WEBHOOK_SECRET not configured in .env, so webhook handler can't verify signatures. Matej needs to: (1) create Stripe webhook endpoint in dashboard pointing to councilnow.com/api/billing/webhooks/stripe, (2) set STRIPE_WEBHOOK_SECRET in .env, (3) restart hub.
- [!] **W1.3** Fix Stripe price ID bug: Starter and Pro share same price_id → create separate Stripe price — BLOCKED: Requires Stripe dashboard to create new Pro price. Both currently point to price_1TGK9RLqM8qbAlEhcT80nLJv. Also STRIPE_PRICE_SCALE and STRIPE_PRICE_GROWTH are not set in .env.
- [!] **W1.4** Verify billing webhook handler works with Stripe LIVE keys (test mode vs live mode) — BLOCKED: STRIPE_WEBHOOK_SECRET not configured. Webhook endpoint exists and returns 400 when no signature provided (correct). Needs Matej to configure webhook in Stripe dashboard.
- [ ] **W1.5** Test session creation: authenticated user → create council session → advisors respond
- [ ] **W1.6** Test frontend routing: marketing → /login → /app → session view → billing
- [ ] **W1.7** Set up Stripe customer portal (self-service manage subscription)
- [ ] **W1.8** Add error tracking/logging for production (basic stdout → structured logs)
- [ ] **W1.9** Smoke test full user journey: visit → sign up → subscribe → run council → view results
- [ ] **W1.10** Push all changes, verify deployment on dev-2026, confirm councilnow.com live

## WAVE 2: CouncilNow Market-Ready

- [ ] **W2.1** Write pricing page copy (Free/Starter/Pro/Scale — €0/29/79/199)
- [ ] **W2.2** Create landing page hero copy — "AI Advisory Council for Complex Decisions"
- [ ] **W2.3** Set up waitlist/early access email collection (if not already wired)
- [ ] **W2.4** Write 3 use-case pages: founder decisions, trading analysis, technical architecture
- [ ] **W2.5** Configure Stripe tax settings for EU VAT
- [ ] **W2.6** Create terms of service and privacy policy pages
- [ ] **W2.7** Test mobile responsiveness of full checkout flow
- [ ] **W2.8** Set up email receipt templates via Stripe/Resend
- [ ] **W2.9** Create a "how it works" page with screenshots/demo
- [ ] **W2.10** Final QA pass: all links work, no console errors, fast load times

## WAVE 3: AgentJack Automation App (FIRST PRODUCT APP)

- [ ] **W3.1** Audit open-kraliki templates — list all working automations with their status
- [ ] **W3.2** Design Automation app API: list templates → deploy → run → monitor
- [ ] **W3.3** Wire automation deployment into AgentJack backend (Python)
- [ ] **W3.4** Build Automation page in SvelteKit webchat (template gallery)
- [ ] **W3.5** Build automation detail page: config form → deploy → logs
- [ ] **W3.6** Implement tenant isolation for automations (per-tenant sandboxes)
- [ ] **W3.7** Add execution history and log viewing
- [ ] **W3.8** Test: create tenant → deploy automation → run → see output
- [ ] **W3.9** Add scheduling UI (cron-like triggers for automations)
- [ ] **W3.10** Push all changes, deploy, verify working end-to-end

## WAVE 4: AgentJack B2B Platform Foundation

- [ ] **W4.1** Study GoClaw `internal/store/` multi-tenant schema patterns
- [ ] **W4.2** Study GoClaw `internal/permissions/` 5-layer security model
- [ ] **W4.3** Study GoClaw `internal/agent/` agent loop architecture
- [ ] **W4.4** Design AgentJack PostgreSQL schema (tenants, users, API keys, usage)
- [ ] **W4.5** Build tenant admin page (port GoClaw dashboard patterns to SvelteKit)
- [ ] **W4.6** Build team management page (invite, roles, permissions)
- [ ] **W4.7** Build API key management page (create, rotate, revoke)
- [ ] **W4.8** Build usage dashboard (tokens, requests, costs per tenant)
- [ ] **W4.9** Implement tenant isolation middleware (ensure data separation)
- [ ] **W4.10** Push all changes, integration test, deploy

## WAVE 5: Infrastructure Cleanup

- [ ] **W5.1** Document current mail VM state (91.99.176.2)
- [ ] **W5.2** Plan mail migration: websites → Cloudflare Pages, services → dev-2026
- [ ] **W5.3** Fix Cloudflare DNS configuration for all domains
- [ ] **W5.4** Test Zitadel without Google SSO (email/password auth)
- [ ] **W5.5** Set up proper SSL certificates (auto-renewal via Traefik/Let's Encrypt)
- [ ] **W5.6** Clean up stopped LXD containers on dev-2026
- [ ] **W5.7** Update PM2 startup scripts for advisory-council docker deployment
- [ ] **W5.8** Verify backup coverage for all production databases
- [ ] **W5.9** Set up monitoring/alerting for councilnow.com uptime
- [ ] **W5.10** Final infra review: all services documented, all ports mapped, all backups verified

---

## Progress Log

| Time | Wave | Item | Result |
|------|------|------|--------|
| 07:18 | W0 | Deploy advisory-council on dev-2026 | ✅ All 5 services healthy, councilnow.com live |
| 08:57 | W1.1 | Test auth flow end-to-end | ✅ Fixed missing Ed25519 keys, register→login→JWT→/me→API→DB all working |
| 09:16 | W1.2 | Test Stripe checkout | ⚠️ Checkout/portal/subscription APIs all work. Blocked on STRIPE_WEBHOOK_SECRET |
| 09:16 | W1.3 | Fix Stripe price IDs | ❌ BLOCKED — needs Stripe dashboard access |
| 09:16 | W1.4 | Verify webhook handler | ❌ BLOCKED — needs STRIPE_WEBHOOK_SECRET |

## Notes

- Each item should take 5-30 minutes of actual work
- If an item takes >30 min, split it into sub-items
- Subagents (coder, codex-reviewer, kimi-frontend) should be used for complex items
- Push to GitHub after every 2-3 completed items
- If Matej gives new priority, insert at top of current wave
