# AgentJack Human Queue UX Execution — 2026-04-26

## Status
✅ Deployed and verified on `https://agentjack-dev.verduona.dev/human`.

## Changes
- Added an inline **Create Human Task** form on `/human` with title, details, category, blocker flag, Add, and Clear controls.
- Human task rows now show details directly in the queue instead of forcing drill-down.
- Row actions now render all available actions instead of only the first quick action.
- Approval tasks expose direct **Start / Approve / Deny / Done** actions from the queue.
- The human task PATCH API now accepts `resolution`, `resolutionNote`, and status updates.
- Task status labels now display human task terms (`open`, `in progress`, `done`) instead of generic section labels.
- Completed tasks are hidden from the active queue by default so smoke/history rows do not clutter execution.
- Dev-2026 deploy checker now treats the direct `agentjack-ag-webui.service` as canonical and skips old reverse-tunnel checks when tunnel ports are not active.

## AgentJack commits
- `2f2fdb5` — Make human queue actionable
- `81be369` — Clarify human queue task status labels
- `d212087` — Show only active human queue tasks

## Local gates
- `cd ag-webui && npm run check` → 0 errors / 0 warnings
- `cd ag-webui && npm run build` → passed
- `pytest tests/test_human_tasks.py tests/test_ag_webui_api_auth.py -q` → 27 passed
- `node scripts/check-ag-webui-write-route-guards.mjs` → passed
- `node scripts/check-ag-webui-error-code-regression.mjs` → passed
- `./scripts/check-local-ag-webui.sh` → passed

## Dev-2026 deploy gates
- Applied latest committed files to `/home/adminmatej/github/agentjack` on `dev-2026`.
- `npm run check` → 0 errors / 0 warnings
- `npm run build` → passed
- write-route guard → passed
- error-code regression → passed
- `systemctl --user restart agentjack-ag-webui.service` → restarted
- `curl http://127.0.0.1:4313/health` → `ok: true`

## Public verification
- `./scripts/check-public-ag-webui.sh` → passed
- `./scripts/check-dev2026-deploy.sh` → passed with only known Telegram inbound continuity warning (`AJ-472513`) and skipped inactive legacy tunnel checks.
- Browser verified `https://agentjack-dev.verduona.dev/human` as signed-in Matej:
  - Create Human Task form visible.
  - Existing approval task shows detail inline.
  - Existing approval task exposes Start, Approve, Deny, Done, Open.
  - Smoke-created task could be added and marked done.
  - Done smoke row is hidden from active queue after final deploy.

## Remaining follow-up
- Telegram inbound continuity still needs a real reply containing pending token `AJ-472513`.
- The old queued approval task remains visible and intentionally not approved/denied by TBA.
