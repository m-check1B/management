# CouncilNow Adaptive Temperature Policy — 2026-04-26

## Outcome

CouncilNow now sets LLM sampling temperature dynamically by analysis stage, inferred task profile, and council role instead of using one flat tier temperature across the whole run.

## Product behavior added

- Added `temperature_policy` to `council-models.json` v13.
- Stage defaults:
  - `round1`: higher exploration (`0.62`) for independent Verbalized Sampling candidates.
  - `round2`: adversarial but controlled (`0.50`).
  - `round3`: lower update/reweighting variance (`0.34`).
  - `synthesis`: deterministic final call (`0.24`).
  - `json_repair`: deterministic (`0.0`).
- Task profiles inferred from the question/context:
  - `creative_strategy`
  - `marketing_or_positioning`
  - `technical_execution`
  - `financial_or_legal`
  - `balanced_decision`
- Role deltas:
  - Strategist / Contrarian / Customer get slightly more exploration.
  - Analyst / Operator / Moderator get lower variance.
- The selected task profile and per-stage temperatures are recorded in `run_metrics.temperature_policy`.
- Savings-router/free-model calls now receive the resolved temperature too.

## Changed files

- `backend/app/services/council_runner.py`
- `backend/app/services/llm_router.py`
- `backend/tests/test_council_model_config.py`
- `council-models.json`

## Commit

- `bc37106 Add adaptive council temperature policy`

## Gates

- `cd backend && .venv/bin/python -m pytest tests/test_council_model_config.py -q` → `7 passed`
- `cd backend && .venv/bin/python -m pytest tests/test_sessions_api.py tests/test_council_runner_json.py tests/test_outcome_verifier_shadow.py tests/test_council_model_config.py -q` → `52 passed`
- `cd frontend && npm run test:smart-questions && node --experimental-strip-types tests/session-report-formatting.test.mjs` → `2 passed` + `4 passed`
- `cd frontend && npm run check` → `0 errors / 0 warnings`
- `cd frontend && npm run build` → passed
- `git diff --check` → passed

## Deploy

- Command: `bash deploy.sh dev-2026`
- Docker image tag: `manual-20260426102045`
- Result: API/hub/frontend containers recreated; `council-api` healthy.

## Public/runtime verification

- `https://councilnow.com/health` → `{"status":"ok","version":"0.4.0","database":"ok"}`
- `https://councilnow.com/app` → HTTP `200`
- `https://councilnow.com/api/billing/status` → `active`; Scale price still `199`
- dev-2026 repo pulled to `bc37106`
- Deployed API runtime config:
  - `runtime_version 13`
  - `temperature_policy_enabled True`
  - `round1_temperature 0.62`
  - `synthesis_temperature 0.24`
  - `creative_strategy_round1_override 0.74`
- Deployed API helper proof:
  - `infer_temperature_task_profile('Design a GTM campaign for CouncilNow')` → `marketing_or_positioning`
  - creative strategist round1 temp → `0.78`
  - technical synthesis temp → `0.14`

## Notes

This complements Verbalized Sampling: early rounds now explore more, adversarial/update phases tighten, and final synthesis stays grounded. No billing, DNS, Stripe, or external outreach changes were made.
