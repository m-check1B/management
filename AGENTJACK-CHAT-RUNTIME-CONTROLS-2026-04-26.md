# AgentJack Chat Runtime Controls — 2026-04-26

## Status

Done. AgentJack chat now exposes accessible model and thinking-level controls directly in the chat runtime bar, including roulette-style cycle buttons.

## Shipped

AgentJack commit: `f62b17e Add chat runtime roulette controls`

Changes:

- Added chat-level runtime model selector with:
  - labelled `<select>`
  - Roulette cycle button
  - Default reset button
  - custom compact `provider/model` input
- Added chat-level thinking-budget selector with:
  - labelled `<select>`
  - Roulette cycle button
  - Default reset button
  - levels: low/fast, balanced/thinking, high/deep
- Persisted per-thread `thinkingOverride` in `chat_store.py`.
- Added CLI command: `chat-thread-thinking`.
- Added SvelteKit API routes:
  - `POST /api/chat/threads/[threadId]/runtime-model`
  - `POST /api/chat/threads/[threadId]/thinking`
- Send/media dispatch now use `currentThread.thinkingOverride || modeProfile.thinking`.
- Dispatch prompt now receives a small runtime-settings preamble so the selected thinking level actually affects behavior, not just UI metadata.
- Updated write-route guard script to recognize `resolveInteractiveSessionUser(...)` and workspace-neutral authenticated voice routes.

## Verification

Local Mac:

- `npm run check && npm run build` in `ag-webui` → passed; `svelte-check found 0 errors and 0 warnings`.
- `pytest tests/test_chat_store.py tests/test_ag_webui_api_auth.py -q` → `24 passed`.
- `node scripts/check-ag-webui-write-route-guards.mjs` → `ok (35 write route files scanned, 3 workspace-optional)`.
- `node scripts/check-ag-webui-error-code-regression.mjs` → `ok (42 route files scanned)`.
- `./scripts/deploy-ag-webui-mac.sh` → local AG WebUI smoke passed.
- Local launchd `com.agentjack.ag-webui` was disabled; re-enabled and started manually.
- `curl http://127.0.0.1:4273/health` → `ok:true`.
- Local `/chat` HTML contains `runtime-model-select`, `thinking-budget-select`, `Roulette`, and `custom provider/model`.

Dev-2026/public:

- Applied commit `f62b17e` files onto `/home/adminmatej/github/agentjack` without disturbing unrelated dirty files on that host.
- Remote `PYTHONPATH=src uv run pytest tests/test_chat_store.py tests/test_ag_webui_api_auth.py -q` → `20 passed`.
- Remote `npm run check` in `ag-webui` → `svelte-check found 0 errors and 0 warnings`.
- Remote `npm run build` in `ag-webui` → build passed.
- Restarted `systemctl --user restart agentjack-ag-webui.service`.
- Remote local health: `curl http://127.0.0.1:4313/health` → `ok:true`.
- Public `/chat` HTML contains `runtime-model-select`, `thinking-budget-select`, `Roulette`, and `custom provider/model`.
- `./scripts/check-public-ag-webui.sh` → public check passed.

## Notes / Caveats

- `./scripts/check-dev2026-deploy.sh` still reports the old SSH reverse-tunnel endpoints missing. Public AG WebUI is healthy through the direct dev-2026 service path, so this is a checker/runbook drift issue, not a blocker for this UI ship.
- The dev-2026 repo has unrelated dirty state, mostly legacy/autodev/webchat files. I applied only this feature's files and did not clean that broader state.
