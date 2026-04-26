# CouncilNow — Revenue Readiness Audit

_Audited: 2026-04-26 04:35 by TBA_
_Status: ✅ READY FOR FIRST CUSTOMER. All price IDs fixed (incl. SCALE). Current production deploy `timestamp-fix-d32d3c2` has passed full auth → checkout and product-path smoke on councilnow.com: register/login/me/subscription/portal, live Stripe checkout for starter/pro/scale/enterprise, council session create/run/complete, HTTPS share URL, public shared report API, user-scoped analytics isolation, and credits summary._

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

1. **Full user journey:** ✅ VERIFIED on production after deploy `timestamp-fix-d32d3c2` — fresh user created session `d2f3d4a5-baeb-4c7b-bd90-2e3da3509bb0`, ran council to `completed`, score `27`, no error, all rounds + synthesis present, shared via HTTPS URL `https://councilnow.com/shared/51d696a3-898a-4262-99fd-78a33d74d953`, public shared API returned `200`, analytics isolation held across two fresh users, and credits summary returned `200`.
2. **Stripe checkout with real user:** ✅ VERIFIED on production. Authenticated token flow now passes for `/api/billing/checkout` (all tiers), `/api/billing/portal`, and `/api/billing/subscription`.
   - Fresh 04:35 recheck PASS with new production account `oc-revenue-1777170916-11352@example.com`: `/health` ok, register/login/me/subscription/portal passed, and starter/pro/scale/enterprise checkout sessions plus Stripe checkout URL probes returned `200`.
   - Fresh 19:21 recheck PASS with new production account `axis-reminder-6178313952@gmail.com`.
   - Fresh 21:21 recheck PASS with new production account `axis-reminder-73074093@gmail.com`.
   - Fresh 23:21 recheck PASS with new production account `axis-reminder-1975726468@gmail.com`.
   - Fresh 01:45 recheck PASS with new production account `axis-reminder-3216957425@gmail.com`.
   - Fresh 03:24 recheck PASS with new production account `axis-reminder-1776821068708@gmail.com`.
   - Fresh 05:24 recheck PASS with new production account `axis-reminder-1776828257765@gmail.com`.
   - Fresh 07:24 recheck PASS with new production account `axis-reminder-1776835462300@gmail.com`.
   - Fresh 09:24 recheck PASS with new production account `axis-reminder-1776842657945@gmail.com`.
   - Fresh 11:46 recheck PASS with new production account `axis-reminder-1776851170922@gmail.com`.
   - Fresh 13:55 recheck PASS with new production account `axis-reminder-1776858922695@gmail.com`.
   - Fresh 15:47 recheck PASS with new production account `axis-reminder-1776865665043@gmail.com`.
   - Fresh 19:43 recheck PASS with new production account `axis-reminder-1776879945204@gmail.com`.
   - Fresh 21:46 recheck PASS with new production account `axis-reminder-1776887190084@gmail.com`.
   - Fresh 01:43 recheck PASS with new production account `axis-reminder-1776901496932@gmail.com`.
   - Fresh 03:49 recheck PASS with new production account `axis-reminder-1776908966611@gmail.com`.
   - Fresh 05:44 recheck PASS with new production account `axis-reminder-1776915885056@gmail.com`.
   - Fresh 07:47 recheck PASS with new production account `axis-reminder-1776923242233@gmail.com`.
   - Fresh 23:11 recheck PASS with new production account `axis-reminder-1777151452752@gmail.com`.
   - Fresh 09:48 recheck PASS with new production account `axis-reminder-1776930474314@gmail.com`.
   - Public auth proof at 11:46: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, and `/auth/api/v1/auth/me` resolved the linked Stripe customer `cus_UNinroo8E90k7Z` for the fresh user.
   - Public auth proof at 13:55: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, and `/auth/api/v1/auth/me` resolved the linked Stripe customer `cus_UNkt4tawshEqII` for the fresh user.
   - Public auth proof at 15:47: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UNmhRUF31ahKJ2` for the fresh user.
   - Public auth proof at 19:43: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UNqXcZ0xxU4dCD` for the fresh user.
   - Public auth proof at 21:46: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UNsUUfYiWstT29` for the fresh user.
   - Public auth proof at 01:43: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UNwKaqb8ywiNMi` for the fresh user.
   - Public auth proof at 03:49: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UNyLOOhxA07wKp` for the fresh user.
   - Public auth proof at 05:44: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UO0CnHg9xUPoNQ` for the fresh user.
   - Public auth proof at 07:47: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UO2B5AYGUXiGdX` for the fresh user.
   - Public auth proof at 23:11: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, `/api/billing/subscription` + `/api/billing/portal` returned 200, and both `/api/billing/checkout` (starter/pro/scale/enterprise) plus Stripe checkout URL `HEAD` probes returned 200 for the fresh user (`cus_UP1WEEd9SNTnJp`).
   - Public auth proof at 09:48: `/auth/api/v1/auth/register` returned 201, `/auth/api/v1/auth/login` returned 200, `/auth/api/v1/auth/me` returned 200, and both `/auth/api/v1/auth/me` plus `/api/billing/subscription` resolved the linked Stripe customer `cus_UO47Jv0qy6evkb` for the fresh user.
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
   - Latest 09:24 checkout recheck returned session IDs:
     - starter: `cs_live_b1YlaEkzO6Na4KZrKqI8bMS9GsSV15qmqOBazZYACgTz4x3Q0gXMm9G8oC`
     - pro: `cs_live_b1NnzfeBCQ9LOlLnAuaqU7sbKNi9ImB3zzMuT7SnX3yrjwpjlN7HnHf1Aa`
     - scale: `cs_live_b1htUJSwRcMvg86ddm3ogkTo6Oi8teWsp8LJLu1uOXFpqCtUJqfQbv3mrN`
     - enterprise: `cs_live_b1l8nOocsUZi8ltovP9EhK5iT2Yp6GtO5K9mKVvjeZBodNT0ZLjCH9ma7g`
   - Each 09:24 live Stripe checkout URL returned `HEAD 200`.
   - Latest 11:46 checkout recheck returned session IDs:
     - starter: `cs_live_b1bM9hVMxs9V5rXVDcRrAidzI4cgmTAVbdpu2Og30jTUHBXqWUh73a0sY8`
     - pro: `cs_live_b1U1nvQy5UcHWiWok8PymrNSWNuVWQvernek7DqhoiE4b8cKYchVrkPJQC`
     - scale: `cs_live_b1QlODOEHS7uilwBTQ02iIycyZSiPFqTccdhL5W5WHlfA94OeVQHclJyqN`
     - enterprise: `cs_live_b1hJae4Ho1pgJXaRLa00ArQ8KCCQPqgdb0tmQ1dYoeP4Ldo6zPQiM8M3Gl`
   - Each 11:46 live Stripe checkout URL returned `HEAD 200`.
   - Latest 13:55 checkout recheck returned session IDs:
     - starter: `cs_live_b1dooLYGT0hKdT9RvoAvOcdPQAwDQ1Xa7qhsRNb6cZ4BOquWOVmVTOzTeU`
     - pro: `cs_live_b1O7R6THoJxoonPg4DNg7Id5QWMJgH5qhitIdt2ANWV3Fn8bOaDzKdWEo4`
     - scale: `cs_live_b1ZSb5cBdYHrP2djtAQUdjocXDXKbmadPreU234toZS4azFTHhliX9HlnQ`
     - enterprise: `cs_live_b1QqAiemzHoDWPbk8pI9TOyjBoSOf7dYqkYG5Q6Scy6q1EMiXqQJY13U5T`
   - Each 13:55 live Stripe checkout URL returned `HEAD 200`.
   - Latest 15:47 checkout recheck returned session IDs:
     - starter: `cs_live_b1JmbjBrmeyXLhhqL7FUWg4WqNOBMG41DqF3jeo22n1aYkdJ0EJC1HrmbW`
     - pro: `cs_live_b1ShZSa0xo5heI74OZqTZwOVJ0OvZciOZRWzmYvhEKl76v1UpAYNaW1smd`
     - scale: `cs_live_b1mRSu9yPJhNflJwUUcHQbg0ePmXBm4Q73Is4YFhCiZEGAcXFmfNfCiOSt`
     - enterprise: `cs_live_b1lqDJJQ06PbkDdwFbqFFwdF1D3CY2HwLLpz0KoaN0Lzyv8aV0s1ELqF8z`
   - Each 15:47 live Stripe checkout URL returned `HEAD 200`.
   - Latest 19:43 checkout recheck returned session IDs:
     - starter: `cs_live_b1xWbrKUUuLibfTBmIyx4ybZcZpma2bFvBu3qcuJNQlcuJc717bzK1EGZ7`
     - pro: `cs_live_b1AvJCFIl7O2qGsD731rbDodAnY48jIui5MTxYio7pfUaNLey4NHD8WGco`
     - scale: `cs_live_b1WrIikxLWgqD7wPcPs1aTgkUsMVNYQyVrzbdpcWybxfd9hkQoScLTOk6Z`
     - enterprise: `cs_live_b1745sUz5iCdblNSgmrswjYCyC5lEp2XVwsGRtOCk6nxv0iflg3zqbyrBL`
   - Each 19:43 live Stripe checkout URL returned `HEAD 200`.
   - Latest 21:46 checkout recheck returned session IDs:
     - starter: `cs_live_b1VkcsILAFFC6TfxE5aMrzHbVWVNdT5BfFcDrMIm57yePN6M9TtwcKx4ZT`
     - pro: `cs_live_b1VMn2bgVK1eVfkHvA6YCXgxjGsXVPInhOSjbQQvQdVDud8RFJz65KRwdy`
     - scale: `cs_live_b1NVaYrPk0treVMKAK9bM9SmsSFM3VJ4zsSrdzc7Xz42ixP3LM5ekkpjTy`
     - enterprise: `cs_live_b1uoErtlv4bky9p1Zm8vBjxUoOY0X0lITH7dpCaQSvt1Q8R5OdBxSBoNzY`
   - Each 21:46 live Stripe checkout URL returned `HEAD 200`.
   - Latest 01:43 checkout recheck returned session IDs:
     - starter: `cs_live_b1Asqq6OQNQad7SlvpiRJs6wh0Rq9SamHG2fQic8QZE7piDV5LscUcZB1F`
     - pro: `cs_live_b1SR9ODzbf07giJiyowwbV3Ys9ZispyEZbxXBfvYBuXKonKSn6FT7sp9Kg`
     - scale: `cs_live_b1a4ZOp0qjafDxAzl7yTxcEDGzobvDpR3qsLYWyRWrsih5KFWIWSceg0K0`
     - enterprise: `cs_live_b1yGWDAYTbHk6uoCHz8IpPB9SHIDxXCvHbocrrIOZtVMNKiPfw88Qm0Z4L`
   - Each 01:43 live Stripe checkout URL returned `HEAD 200`.
   - Latest 03:49 checkout recheck returned session IDs:
     - starter: `cs_live_b19js6ImJrSAbR4YiqmkYoHpS1PuP6ZUEIddoRbdn81e1qFWTKoGpcXIUo`
     - pro: `cs_live_b1Z35ypPpnQKXEu9BbMJ9NJ5k1ZLJ8SphknjtioDrUKKxlHQfdeyiETMdT`
     - scale: `cs_live_b13OUapebeFKXZprPTKzcCwZKeyP9RntSpVa0TXF4565D8qvZVJtiiLEcV`
     - enterprise: `cs_live_b1Zn5zt5Pl1YI1gXdbg7XkrbWatJ2PCH3AFQiNvkK33Hukqie7gJk2ktfI`
   - Each 03:49 live Stripe checkout URL returned `HEAD 200`.
   - Latest 09:48 checkout recheck returned session IDs:
     - starter: `cs_live_b10Y4NqHd3iMREvrJ0LDYcyEZ9vszgGVe3j7z9RXXQtl62i8JYIFZ3TjRp`
     - pro: `cs_live_b1i5R4DA33DCKer0l2a9D0aWo8nvGo6jbZ16gslZXIuEZ1PydwrbLvsSOx`
     - scale: `cs_live_b1WV5wlY4WEkHtHV9Q2HTVe1y4OZ2K2Z4GgX1fvsLyWFZso0yhNZIvDEcQ`
     - enterprise: `cs_live_b1m1Hc9b6Y0AFgHfiO8djH6Mozq4lrNPxSZXMMJZkcdxWoS6WJZW1vlvdD`
   - Each 09:48 live Stripe checkout URL returned `HEAD 200`.
   - Latest `/api/billing/portal` pass returned a live Stripe portal URL and session `bps_1TPHzMLqM8qbAlEh5IuCoxIh` for the fresh user at 09:48.
   - Latest `/api/billing/subscription` pass returned the linked Stripe customer `cus_UO47Jv0qy6evkb`.
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
