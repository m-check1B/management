# CouncilNow Verbalized Sampling Wiring — 2026-04-26

## Outcome

CouncilNow now uses Verbalized Sampling in the council protocol instead of collapsing each advisor/synthesis step to a single generic answer.

## Product behavior added

- Round 1: each advisor must generate 5 candidate predictions for each option via `candidate_distribution`, assign probabilities summing to 100, include at least one 5–20% non-obvious tail candidate, and capture `tail_insight`.
- Round 2: attacks must treat candidate distributions/tail insights as first-class prediction material and challenge over/underweighted tail candidates.
- Round 3: advisors must reweight candidate distributions after attacks via `candidate_distribution_update` and preserve a `tail_risk_or_opportunity`.
- Synthesis: final report must produce 5 probabilistic final decision calls in `recommendation_distribution` and add `tail_consideration`.
- Normalization: if synthesis omits explicit `recommendation` / `recommendation_option`, the app infers them from the highest-probability recommendation-distribution entry.
- Reports/export: Verbalized Sampling distributions render as readable markdown, not raw JSON.

## Changed files

- `backend/app/services/council_runner.py`
- `backend/tests/test_council_model_config.py`
- `council-models.json`
- `frontend/src/lib/session-report.ts`
- `frontend/tests/session-report-formatting.test.mjs`

## Commit

- `c599b30 Add verbalized sampling to council prompts`

## Gates

- `cd backend && .venv/bin/python -m pytest tests/test_council_model_config.py -q` → `6 passed`
- `cd backend && .venv/bin/python -m pytest tests/test_sessions_api.py tests/test_council_runner_json.py tests/test_outcome_verifier_shadow.py tests/test_council_model_config.py -q` → `51 passed`
- `cd frontend && npm run test:smart-questions && node --experimental-strip-types tests/session-report-formatting.test.mjs` → `2 passed` + `4 passed`
- `cd frontend && npm run check` → `0 errors / 0 warnings`
- `cd frontend && npm run build` → passed
- `git diff --check` → passed

## Deploy

- Command: `bash deploy.sh dev-2026`
- Docker image tag: `manual-20260426095837`
- Result: `council-api`, `council-hub`, and `council-frontend` recreated/restarted successfully; API container reported healthy.

## Public/runtime verification

- `https://councilnow.com/health` → `{"status":"ok","version":"0.4.0","database":"ok"}`
- `https://councilnow.com/app` → HTTP `200`
- `https://councilnow.com/api/billing/status` → current pricing route still intact: Starter €29 / Pro €79 / Scale €199 / Enterprise €499
- Runtime config inside deployed API container:
  - `version 12`
  - `round1_has_vs True`
  - `round2_has_distribution True`
  - `round3_has_reweight True`
  - `synthesis_has_distribution True`

## Notes

This is intentionally a prompt/protocol-level upgrade plus report formatting support. It does not require billing, DNS, Stripe, or external outreach changes.
