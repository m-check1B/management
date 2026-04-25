# CouncilNow First-Customer Product-Quality Triage — 2026-04-26

## Verdict

**Status: billing-path verified / product path functional, but NOT first-customer-ready yet.**

The core customer path can work: a fresh account can register/login, create a council session, run it to completion, retrieve history, and publish a shared report. However, the triage found two launch blockers that should be fixed or explicitly accepted as pilot limitations before claiming first-customer readiness.

## Evidence Summary

Production base: `https://councilnow.com`

### Passed

- **Health:** `GET /health` returned `200` with `{"status":"ok","database":"ok"}`.
- **Auth API:** fresh local test user registered and logged in successfully.
  - `POST /auth/api/v1/auth/register` -> `201`
  - `GET /auth/api/v1/auth/me` -> `200`
  - `POST /auth/api/v1/auth/login` -> `200`
- **Council session API:** fresh test user created and ran an Efficient session to terminal completion.
  - session id: `91bbd700-3ac4-4456-bc99-a82e6591ee9d`
  - create: `200`, status `draft`
  - run: `200`, status `running`
  - poll terminal: `200`, status `completed`, `score=27`, `error_message=null`
  - output included `round1_predictions`, `round2_attacks`, `round3_updates`, and `synthesis`.
- **History:** authenticated `GET /api/v1/council/sessions/?limit=5` returned the test user's completed session.
- **Credits:** authenticated `GET /api/v1/council/sessions/credits/summary` returned a sane free-plan credit summary: 6 included monthly credits, 1 used, 5 remaining; Efficient costs 1 credit.
- **Public share:** `POST /api/v1/council/sessions/{id}/share` returned a share token; public `GET /api/v1/council/sessions/shared/{token}` returned a share-safe payload.
  - share token: `0e141c92-98f7-4eac-87f2-fa3346c24524`
  - public page: `https://councilnow.com/shared/0e141c92-98f7-4eac-87f2-fa3346c24524` returned `200 text/html`.
- **Browser UI:** a signed-in Google browser profile reached `/app/sessions/new`, created a Guided Setup session, and rendered the live debate screen.
  - UI-created session id: `a4b0fa25-cab1-462d-b877-fdbd48125593`
  - Evidence before browser-control timeout: live page showed status `Running`, tier `EFFICIENT`, Round 1 active.
- **Frontend local gate:** `npm run check && npm run build` passed with 0 Svelte diagnostics.
- **Backend runner/quality gates:** `pytest tests/test_council_runner_json.py tests/test_outcome_verifier_shadow.py -q` passed: `23 passed`.
- **Hub billing/auth tests:** `pytest tests/test_billing_centralized.py tests/test_register_login.py -q` in `backend/hub-fork` passed: `37 passed`.

### Evidence files from this triage

Local unsanitized runtime evidence files, intentionally not committed because they are test-run artifacts:

- `/tmp/councilnow-quality-smoke-1777156303.json`
- `/tmp/councilnow-quality-followup.json`
- `/tmp/councilnow-quality-history.json`

They contain no access tokens, but they do contain generated test account identifiers and live session/share ids.

## Findings

### BLOCKER 1 — Authenticated analytics endpoint is not user-scoped

**Impact:** Any authenticated user can see aggregate analytics across all sessions, not just their own. This is a privacy/tenant-boundary issue and also contaminates per-user conversion metrics.

**Evidence:**

- Fresh test user history: `GET /api/v1/council/sessions/?limit=5` -> `session_count=1`, first status `completed`.
- Same test user's analytics: `GET /api/v1/council/sessions/analytics?window=all` -> `total_sessions=43`, `completed=23`, `failed=4`.

**Root cause:** `backend/app/routers/sessions.py:get_session_analytics` accepts `user_id = Depends(get_current_user_id)`, but the query selects all `CouncilSession` rows and does not filter by `CouncilSession.user_id == user_id`.

**Required fix:** Add the user filter, add regression tests for per-user isolation, and verify a fresh user sees only their own analytics.

**Classification:** Launch blocker before first-customer readiness claim.

---

### BLOCKER 2 — Guided Setup can ask irrelevant follow-up questions

**Impact:** First-customer UX can feel obviously broken. In the browser UI test, a decision about CouncilNow pilot readiness was classified as a hiring/team decision and asked hire-specific questions.

**Evidence from production UI snapshot:**

Prompt entered:

> Should CouncilNow run a controlled first-customer pilot next week while the team fixes product-quality bugs, or wait until all launch polish is complete?

Follow-up questions shown:

1. `What's your budget for this hire?`
2. `What would this person do in their first week?`
3. `What happens if you DON'T hire?`
4. `What would your biggest customer say about this?`

**Root cause:** `frontend/src/routes/app/sessions/new/+page.svelte:generateSmartQuestions` uses naive keyword matching. The condition `lower.includes('hire') || lower.includes('team')` routes any prompt containing `team` into the hiring-question branch.

**Required fix:** Replace or harden this heuristic. Minimum safe patch: remove bare `team` as a hiring trigger and add category-specific tests. Better fix: use the backend `generate_session_structure` preflight or a small server-generated follow-up-question endpoint so questions are grounded in the actual decision type.

**Classification:** Launch blocker unless first customer is manually guided and warned.

---

### PILOT RISK — Shared-report URL is generated as `http://` behind HTTPS production

**Impact:** Copied share links look insecure/unpolished. The HTTP URL does redirect to HTTPS, so the link works, but this should not ship in a polished customer handoff.

**Evidence:**

- Share API returned `share_url: "http://councilnow.com/shared/0e141c92-98f7-4eac-87f2-fa3346c24524"`.
- Direct HTTP request redirects with `302` to `https://councilnow.com/shared/...`.

**Root cause:** `backend/app/routers/sessions.py:build_share_url` uses `request.base_url`; Traefik/proxy forwarded scheme is not being trusted/applied for this generated URL.

**Required fix:** Generate share URLs from canonical `APP_URL` or ensure forwarded proto middleware/config is correct.

**Classification:** Pilot-acceptable if links are manually checked/replaced; fix before broad sales/organic sharing.

---

### INTERNAL QA BLOCKER — `backend/tests/test_sessions_api.py` is stale against auth-required sessions API

**Impact:** The local session API test suite is no longer a reliable regression gate. Most failures are `401` because tests create/run sessions without auth while production routes now require `get_current_user_id`.

**Evidence:**

Command:

```bash
cd /Users/matejhavlin/github/applications/advisory-council-app/backend
.venv/bin/python -m pytest tests/test_sessions_api.py tests/test_council_runner_json.py tests/test_outcome_verifier_shadow.py -q
```

Result:

- `20 failed, 23 passed`
- Failing session tests return `401` for `/api/v1/council/sessions/` and related share/history paths.

**Root cause:** `backend/tests/conftest.py` creates a bare `TestClient` with no authenticated user/token helper, while `backend/app/routers/sessions.py` now requires auth on create/list/get/share/run/analytics.

**Required fix:** Add an authenticated test helper/fixture, then restore session API tests as a meaningful release gate.

**Classification:** Internal QA blocker, not directly customer-facing.

---

### OBSERVATION — Efficient production run completed but took around 5 minutes

**Impact:** This may be acceptable for a decision sprint, but first customer onboarding should set expectation clearly. The live UI does show a progress monitor and checks for results every 3 seconds.

**Evidence:** API smoke reached terminal completion on poll 30 with 10-second poll intervals.

**Classification:** Pilot-acceptable; measure across more runs before tuning.

## Readiness Classification

| Area | Result | Notes |
|---|---:|---|
| Marketing/pricing pages | ✅ Pass | `/app/pricing`, `/app/sessions/new`, shared page render as HTML. |
| Auth/billing API | ✅ Pass | Register/login/me + billing tests pass. |
| Session create/run | ✅ Pass | Fresh account completed Efficient session. |
| Result quality | ✅ Pass with caveat | Sample recommendation was coherent prose with structured synthesis. |
| Public share | ⚠️ Risk | Works, but generated URL uses `http://`. |
| Guided Setup UX | ❌ Blocker | Irrelevant hire questions for non-hiring prompt. |
| Analytics isolation | ❌ Blocker | Authenticated user sees global aggregate analytics. |
| Regression tests | ⚠️ Internal blocker | Sessions tests need auth fixture repair. |

## Linear Updates

- `KRA-3674` — triage parent marked Done with evidence comment.
- `KRA-3675` — `[CouncilNow][P0] Scope session analytics to authenticated user`.
- `KRA-3676` — `[CouncilNow][P0] Fix Guided Setup irrelevant follow-up questions`.
- `KRA-3677` — `[CouncilNow][P1] Generate HTTPS canonical shared-report URLs`.
- `KRA-3678` — `[CouncilNow][P1] Restore authenticated session API regression tests`.
- `KRA-3673` — sales/outreach issue commented: keep external outreach internal-only until blockers are fixed or founder-accepted.

## Decision

Do **not** claim CouncilNow is first-customer-ready yet.

Safe wording remains:

> CouncilNow is billing-path verified and sales-prep ready. The product path is functional in production, but first-customer readiness is blocked by product-quality issues found in the 2026-04-26 triage.

## Recommended execution order

1. Fix analytics user scoping and add isolation regression tests.
2. Fix Guided Setup follow-up generation and add tests around non-hiring prompts containing `team`.
3. Fix canonical HTTPS share URL generation.
4. Repair session API tests with authenticated fixtures.
5. Re-run production smoke: Google login -> guided setup -> run -> history -> share -> public memo.
6. Only then unblock external first-customer outreach from `KRA-3673`.
