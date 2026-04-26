# Priority Execution — 2026-04-26

## Founder Priority Lock

Today has two lanes, in order:

1. **CouncilNow first.** Fix all bugs, flows, pricing, coupon workflow, and model routing.
2. **AgentJack second.** Complete the new version and internally deploy/verify it at `https://agentjack-dev.verduona.dev`.

## Guardrails

- CouncilNow first-customer outreach remains held until product, pricing, coupon flow, and sender/report loops are green.
- No payments/billing-plan mutations are performed without explicit founder approval; code/config/checkout/coupon verification is allowed.
- Deploy only through canonical build → deploy → verify lanes.
- Every completed lane needs evidence: tests, public smoke, E2E proof, commit, and deploy verification.

## CouncilNow Acceptance Criteria

- Public auth/login/signup still works.
- Session creation/run/history/share still works.
- Pricing displayed in UI matches backend billing tiers and Stripe configuration.
- Checkout works for all active tiers.
- Coupon/promo workflow is implemented or repaired and verified end-to-end.
- Model routing is current, stable, and report-safe.
- Report export/shared pages do not leak raw JSON/schema/model metadata.
- Evidence recorded in management repo.

## AgentJack Acceptance Criteria

- Latest AG WebUI/runtime changes are on `main` and deployed to dev-2026 internal surface.
- `https://agentjack-dev.verduona.dev/health`, `/readyz`, and `/chat` are green.
- New chat runtime model/thinking controls are present publicly.
- Critical flows work: chat send, runtime dispatch, tasks/human tasks where applicable, live voice guard remains secure.
- Deployment/runbook drift is classified and either fixed or recorded as non-blocking with exact reason.
- Evidence recorded in management repo.

## Current Execution Plan

1. Audit CouncilNow code/config/UI for pricing/coupon/model/report flow.
2. Implement CouncilNow fixes with focused tests.
3. Deploy CouncilNow and rerun dummy-customer/public auth→checkout→coupon→session proof.
4. Audit AgentJack new version/deploy surface.
5. Complete remaining AgentJack gaps, deploy to dev-2026, and rerun public/internal checks.
