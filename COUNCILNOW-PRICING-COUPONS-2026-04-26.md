# CouncilNow Pricing/Coupon Execution — 2026-04-26

## Status
✅ Code fixed, committed, deployed to `https://councilnow.com`, and verified at health/pricing-status level.

## Changes
- Pricing/status compatibility route now matches current pricing:
  - Starter €29/month
  - Pro €79/month
  - Scale €199/month
  - Enterprise €499/month
- Added Scale tier to legacy centralized billing tier enum/prices.
- Pricing UI now forwards `coupon_code` and `allow_promotion_codes` for credit top-up checkout, not only subscription checkout.
- Added regression coverage for plan checkout coupon forwarding, credit checkout coupon forwarding, and current displayed backend pricing.

## Commit
- Advisory Council app: `2224a3f Fix CouncilNow pricing and coupon checkout flows`

## Local gates
- `backend/hub-fork/.venv/bin/python -m pytest tests/test_billing_centralized.py -q` → 26 passed
- `backend/.venv/bin/python -m pytest tests/test_sessions_api.py tests/test_council_runner_json.py tests/test_outcome_verifier_shadow.py tests/test_council_model_config.py -q` → 49 passed
- `frontend npm run test:smart-questions` → 2 passed
- `frontend npm run check` → 0 errors / 0 warnings
- `frontend npm run build` → passed

## Deploy
- Ran `bash deploy.sh dev-2026` from `/Users/matejhavlin/github/applications/advisory-council-app`.
- Docker images rebuilt and loaded as `manual-20260426092126`.
- Containers recreated:
  - `advisory-council-council-api-1`
  - `advisory-council-council-hub-1`
  - `advisory-council-council-frontend-1`

## Public verification
- `https://councilnow.com/health` → `status: ok`, `database: ok`
- `https://councilnow.com/api/billing/status` → returned Starter 29 / Pro 79 / Scale 199 / Enterprise 499
- `https://councilnow.com/app/pricing` → HTTP 200

## Remaining blocker
- Live coupon E2E still needs an existing safe Stripe coupon/promotion code approved for verification. No Stripe dashboard mutation was performed.
