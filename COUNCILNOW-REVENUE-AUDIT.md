# CouncilNow — Revenue Readiness Audit

_Audited: 2026-04-19 07:43 by Axis_
_Status: 90% ready. Missing: one real end-to-end test with a real user_

## What's Working ✅

| Component | Status | Evidence |
|-----------|--------|----------|
| Website (councilnow.com) | ✅ LIVE | Cloudflare, HTTPS, 200 OK |
| Landing page | ✅ LIVE | CTA buttons pointing to /app |
| App API (dev-2026:8210) | ✅ HEALTHY | `{"status":"ok","version":"0.4.0","database":"ok"}` |
| App frontend (/app) | ✅ LIVE | SvelteKit SSR, 200 OK |
| Pricing page | ✅ LIVE | 4 tiers: Free, €29/mo, €79/mo, €199/mo |
| Login page | ✅ LIVE | Zitadel OIDC wired |
| Stripe keys | ✅ LIVE | `sk_live_` + `pk_live_` in hub-fork/.env |
| Stripe checkout API | ✅ REACHES STRIPE | `/api/billing/checkout` — returned "No such customer: cus_test" (means Stripe connection works) |
| Stripe prices configured | ✅ | Starter/Pro/Enterprise price IDs set |
| Customer portal | ✅ BUILT | `/api/billing/portal` endpoint exists |
| Credits purchase | ✅ BUILT | `/api/billing/checkout/credits` endpoint |
| Zitadel SSO | ✅ CONFIGURED | identity.verduona.dev, Google working |
| Auth → billing flow | ✅ WIRED | JWT → customer_id → Stripe checkout |
| Webhooks | ✅ BUILT | billing webhook handler exists |
| Session history | ✅ BUILT | /app/sessions route |
| New session flow | ✅ BUILT | /app/new-session route |
| i18n | ✅ BUILT | English (others via $t keys) |
| Legal pages | ✅ BUILT | /legal/privacy, /legal/terms |
| PostHog analytics | ✅ WIRED | EU instance, capturing |
| SEO | ✅ BUILT | Schema.org, OG tags, meta descriptions |

## What Needs Testing 🔍

1. **Full user journey:** Sign up with Google → create session → run council → see results
2. **Stripe checkout with real user:** Auth → get customer ID → checkout → Stripe page → success redirect
3. **Webhook delivery:** Stripe → councilnow.com webhook endpoint → subscription activated
4. **Credits purchase flow:** Buy credits → credits appear in account → use for session
5. **Zitadel without Google:** Email/password signup

## Pricing (Already Set)

| Tier | Price | Description |
|------|-------|-------------|
| Free | €0 | Limited sessions |
| Starter | €29/mo | Solo operators, weekly recommendations |
| Pro | €79/mo | Teams, deeper synthesis, priority queue |
| Scale | €199/mo | Leadership teams, cross-session memory |

## What I'm Doing Next

1. **Test the full auth → checkout flow** with a test user
2. **Verify webhook endpoint** is reachable from Stripe
3. **If anything breaks, fix it**
4. **Report to Matej: "Ready for first customer" or "Here's what's broken"**

## Blockers Found

- **Starter and Pro use the SAME Stripe price ID** (`price_1TGK9RLqM8qbAlEhcT80nLJv`). This means €29 Starter and €79 Pro would charge the same amount. Need to check if this is intentional (credits-based) or a config bug.
- **Need Matej to confirm:** Is the pricing model credit-based (same price ID, different credit amounts) or subscription-tier-based (needs separate price IDs)?
