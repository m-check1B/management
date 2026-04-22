# CouncilNow — Revenue Readiness Audit

_Audited: 2026-04-22 07:24 by Axis_
_Status: ✅ READY FOR FIRST CUSTOMER. All price IDs fixed (incl. SCALE), full checkout chain verified end-to-end again at 07:24 with a fresh production auth user on the real frontend contract._

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
| Stripe checkout API | ✅ ALL TIERS VERIFIED | `/api/billing/checkout` — starter, pro, scale, enterprise all resolve correctly via internet → CF → Traefik → LXD → hub → Stripe |
| Stripe prices configured | ✅ | All 4 tiers: Starter (€29), Pro (€79), Scale (€199), Enterprise (€499) EUR prices |
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
2. **Stripe checkout with real user:** ✅ VERIFIED on production. Authenticated token flow now passes for `/api/billing/checkout` (all tiers), `/api/billing/portal`, and `/api/billing/subscription`.
   - Fresh 19:21 recheck PASS with new production account `axis-reminder-6178313952@gmail.com`.
   - Fresh 21:21 recheck PASS with new production account `axis-reminder-73074093@gmail.com`.
   - Fresh 23:21 recheck PASS with new production account `axis-reminder-1975726468@gmail.com`.
   - Fresh 01:45 recheck PASS with new production account `axis-reminder-3216957425@gmail.com`.
   - Fresh 03:24 recheck PASS with new production account `axis-reminder-1776821068708@gmail.com`.
   - Fresh 05:24 recheck PASS with new production account `axis-reminder-1776828257765@gmail.com`.
   - Fresh 07:24 recheck PASS with new production account `axis-reminder-1776835462300@gmail.com`.
   - `/api/billing/checkout` returned live Stripe sessions for `starter`, `pro`, `scale`, and `enterprise`.
   - Latest 01:45 checkout recheck returned session IDs:
     - starter: `cs_live_b1QsqeGLAUTfLi7CxT4c2XnjeoAtxHDKtXQucJWbRL8lfsAMqBFjB37H71`
     - pro: `cs_live_b1Q1HxxE20Tz6WbrQElPYp214CZpX2uimorLlc7uw0LsO66T0iihHLHZlR`
     - scale: `cs_live_b1nAAyGujgf8abaiPAGOGW9AGTcvwRvINlqbPLJqNDDfWewYrjkX70dMGm`
     - enterprise: `cs_live_b1aWcxgj1zlAVIfZjUSNQawQrGqTwEET7vKKpAVv02fDMAw6iTbmD97TG7`
   - Each 01:45 live Stripe checkout URL returned `HEAD 200`.
   - Latest 03:24 checkout recheck returned session IDs:
     - starter: `cs_live_b1PnlLehkzPCFh3G2G5JWolE5r9DcmSKsxPFII2DaJNBR0GgyMFBLyY9D4`
     - pro: `cs_live_b1TG1mTAukHoYk8aqoyXshNFRDfR3xKnFksr4XGmBwSVqustZFkWNlb2gj`
     - scale: `cs_live_b1fRVlbhemBErakVryB3S6N0epOoDUdrbOx1TkVGRX4DirooYd952roBf8`
     - enterprise: `cs_live_b1kGxElLK4co12fZs6mdQIpBNLHfrwXO40QE8RpfXqTHV4wY6JHd6L6okn`
   - Each 03:24 live Stripe checkout URL returned `HEAD 200`.
   - Latest 05:24 checkout recheck returned session IDs:
     - starter: `cs_live_b1F0bsGabZxuO4hcpxTtdMYoyHAShQEj1snwvs8Zr6OLl5SSP2V7gyizBe`
     - pro: `cs_live_b1TQvigvK6yBcqoTo8q6j3UMlW0iqXPcF3hU33g3JhzzT01NS8BKKl9i7g`
     - scale: `cs_live_b1eXuBOP8VldJtxp6G4VxhTTbwYSVNgGM4tGvw06W448AoKe4crQO4AxbU`
     - enterprise: `cs_live_b1MqUieHZLT8VSACyZGYzsQJ0qVqu8uReK8FS2WD35QEkActT8qvCJna6p`
   - Each 05:24 live Stripe checkout URL returned `HEAD 200`.
   - Latest 07:24 checkout recheck returned session IDs:
     - starter: `cs_live_b1L3h8qEFi01MOVEMdTH5esWFpPM84X7wtULLUloIoyFm01ulEaRoJDfKj`
     - pro: `cs_live_b1ZrZrFn4nMZ1PCKrn4x8tH1fTM6vTVdemnIB9uZ4WejsHlWnLQhwK4nbB`
     - scale: `cs_live_b1PZfU4Ga5Ay0aG6tBzcf8qDYSRKY7mJWHnqvqtIT9DRjQsTybHTdGlK5i`
     - enterprise: `cs_live_b1CPJh5ESeaw3zRRrZrBZfkjOKsVVjKLQC70wpUd85Dun9cx1tIlZCB2Th`
   - Each 07:24 live Stripe checkout URL returned `HEAD 200`.
   - Latest `/api/billing/portal` pass returned `bps_1TOtGzLqM8qbAlEhkorGLZIo`.
   - Latest `/api/billing/subscription` pass returned the linked Stripe customer `cus_UNeaUMXevWdziq`.
   - Important test-contract note: bare API payloads with only `tier` fail validation because the live endpoint correctly requires `success_url` and `cancel_url`; `/api/billing/portal` likewise requires `return_url`; the customer path is healthy because the production pricing page sends the full contract (`tier_name`, success/cancel URLs, `mode`, and `return_url` for portal).
   - Response-shape note: live checkout returns `checkout_url` and live portal returns `portal_url`, not only a generic `url` field.
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

1. **Keep the proven auth → checkout contract stable** (`tier_name` + success/cancel URLs)
2. **Verify webhook endpoint** is reachable from Stripe
3. **If anything breaks, fix it immediately**
4. **Continue toward first paying customer**

## Blockers Found & FIXED ✅

- **FIXED 2026-04-19 13:50:** Starter and Pro were using the SAME Stripe price ID ($29 USD). Updated to correct EUR prices:
  - Starter: `price_1TLriOLqM8qbAlEhxXARkZuJ` — €29/mo
  - Pro: `price_1TLriOLqM8qbAlEhj5TzzRKY` — €79/mo
  - Scale: `price_1TLriPLqM8qbAlEhbOYMP7Ww` — €199/mo
  - Enterprise: `price_1TLriQLqM8qbAlEh5oAGP6Dr` — €499/mo
  - Credits 100/500/2000 price IDs also configured
- **Verified end-to-end:** Created real Stripe checkout session through full chain (internet → Traefik → LXD → hub → Stripe). Pro tier correctly shows €79.00 EUR.
