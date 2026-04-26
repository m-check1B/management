# CouncilNow dev-server refresh deploy — 2026-04-26

## Scope
- Published CouncilNow marketing + advisory-council-app through the existing dev-2026/LXD deployment path.
- Target: `dev-2026` → LXD `advisory-council` → `/opt/advisory-council` Docker Compose stack.
- Deploy tag: `councilnow-refresh-20260426204024`.
- App commit pushed after deploy: `124ad72 Polish CouncilNow app experience`.

## Build gates
- `websites/councilnow.com`: `pnpm check` → 0 errors / 0 warnings.
- `websites/councilnow.com`: `pnpm build` → passed; known SvelteKit static prerender warnings for live app routes (`/app`, `/app/pricing`, `/legal/privacy`, `/legal/terms`) because those are Traefik-routed to the app, not the marketing container.
- `advisory-council-app/frontend`: `npm run check` → 0 errors / 0 warnings.
- `advisory-council-app/frontend`: `npm run build` → passed.
- `advisory-council-app`: `docker compose config -q` → passed.
- `git diff --check` → passed after fixing one trailing whitespace in `frontend/src/routes/app/+layout.svelte`.

## Deploy
- Ran canonical helper from `/Users/matejhavlin/github/websites/councilnow.com`:
  - `DEPLOY_TAG=councilnow-refresh-20260426204024 ./scripts/deploy-to-production.sh dev-2026`
- Helper rendered CouncilNow marketing with live PostHog config, called `advisory-council-app/deploy.sh`, built images on `dev-2026`, pushed them into LXD, and ran `docker compose up -d` in `/opt/advisory-council`.
- New images running:
  - `advisory-council-api:councilnow-refresh-20260426204024`
  - `advisory-council-hub:councilnow-refresh-20260426204024`
  - `advisory-council-frontend:councilnow-refresh-20260426204024`

## Live verification
- Marketing verify script passed:
  - `pnpm verify:live`
  - `https://councilnow.com/` 200
  - PostHog config present, host `https://eu.i.posthog.com`
  - CTA locations: `nav, hero, mid, closing, footer`
  - CTA labels: `start_first_council, review_sprint`
- Public route checks:
  - `https://councilnow.com/` → 200, `60618` bytes, title `CouncilNow — Decision Sprints For Hard Business Calls`
  - `https://councilnow.com/health` → 200, `{"status":"ok","version":"0.4.0","database":"ok"...}`
  - `https://councilnow.com/app` → 200, title `Advisory Council — Decision Sprints For Hard Business Calls`
  - `https://councilnow.com/login` → 200, title `Advisory Council — Decision Sprints For Hard Business Calls`
- Container health after deploy:
  - `council-api` healthy
  - `council-hub` healthy
  - `council-frontend` running
  - `council-router` healthy
  - `open-notebook` healthy
  - `council-db` healthy
  - `council-marketing` running
- Auth/core smoke:
  - fresh register → 201
  - login → 200
  - `/auth/api/v1/auth/me` → 200
  - authenticated draft session creation at `/api/v1/council/sessions/` → 200
  - smoke session: `db22c224-7798-438b-85fe-0c580eaaf680`, status `draft`, tier `efficient`

## Notes / caveat
- Current live marketing contract still uses `websites/councilnow.com/index.html` as the primary deployed source. Dirty SvelteKit marketing source under `websites/councilnow.com/src/` builds locally, but it is not wired into the live marketing container; deploying it directly would require a routing/assets change because generated `/_app` assets currently conflict with the app frontend Traefik route. I did not change that architecture during this refresh.
- Public Python `urllib` smoke was blocked by Cloudflare 1010 due browser-signature rules; retrying with a browser-like curl user-agent passed.

## 20:55 Follow-up redeploy — Svelte marketing artifact

- Deploy tag: `councilnow-webapp-20260426205139`
- Scope: rebuilt and republished `websites/councilnow.com/build/` as the `council-marketing` static artifact, plus current `advisory-council-app` API/hub/frontend images.
- Build evidence:
  - `councilnow.com`: `pnpm check` passed, `pnpm build` passed.
  - `advisory-council-app/frontend`: `pnpm check` passed, smart questions/evidence-notebook/session-report tests passed, `pnpm build` passed.
  - `advisory-council-app/backend`: targeted app backend suite `70 passed` with Python 3.13 temp venv.
- Deploy evidence:
  - `council-api`, `council-hub`, and `council-frontend` recreated with tag `councilnow-webapp-20260426205139`.
  - `council-api` and `council-hub` healthy after warmup.
  - Fixed static marketing permissions after Apache initially returned 403; patched `scripts/deploy-on-dev-2026.sh` so future guest extraction runs `chmod -R a+rX marketing`.
- Public verification:
  - `https://councilnow.com/` → 200, new Svelte CTA marker present, old static title absent, Svelte `_app/immutable` assets linked.
  - `https://councilnow.com/_app/version.json` → 200 (`1777229516621`).
  - `https://councilnow.com/app` → 200, app shell assets present.
  - `https://councilnow.com/health` → 200, `{"status":"ok","version":"0.4.0","database":"ok"}`.
  - `https://councilnow.com/login` → 200, login shell assets present.
- Browser proof: root rendered the new “Open one hard decision. Get six expert advisors working in five minutes” page; primary CTA navigated into the Quick Sprint app screen.

## 21:00 Final deploy — analytics contract restored

- Final deploy tag: `councilnow-webapp-analytics-20260426205838`.
- Restored the Svelte marketing artifact's analytics/readiness contract before final publish:
  - `window.COUNCILNOW_POSTHOG_CONFIG` present in live HTML.
  - Live PostHog key injected at deploy time; placeholder absent from public HTML.
  - `data-ph-event="cta_clicked"` markers present for nav, hero, closing, and footer CTAs.
  - `scripts/verify-live-marketing.mjs` now checks the Svelte `build/index.html` artifact and current `src/lib/i18n/en.json` copy.
- Final verification evidence:
  - `pnpm verify:live` passed.
  - Public `https://councilnow.com/` 200; new CTA present; PostHog config and CTA tracking attrs present; no placeholder key.
  - Public `https://councilnow.com/_app/version.json` 200.
  - Public `https://councilnow.com/app` 200 with app shell assets.
  - Public `https://councilnow.com/health` 200 with `status=ok`, `version=0.4.0`, `database=ok`.
  - Public `https://councilnow.com/login` 200 with login shell assets.
  - Browser proof: marketing CTA navigated to the Quick Sprint app screen.
