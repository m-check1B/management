# AG WebUI voice route guard evidence — 2026-04-26

## Scope
- Repo: `/Users/matejhavlin/github/AgentJack`
- App: `ag-webui`
- Routes reviewed:
  - `POST /api/chat/transcribe`
  - `POST /api/chat/live-voice/session`

## Policy decision
- Treat both routes as **write-like / capability-minting**.
- They must require **real product auth** (trusted proxy headers or signed session cookie).
- **Env-auth fallback is not sufficient** for these routes, because it effectively lets an anonymous browser inherit operator identity when `AG_WEBUI_ALLOW_ENV_AUTH=true`.
- I did **not** add a new anonymous continuation path.
- I also did **not** find an existing narrow signed/ephemeral AG WebUI voice-session continuation guard in this codebase. Current live-voice flow is:
  1. authenticated POST to `/api/chat/live-voice/session`
  2. server mints xAI client secret
  3. browser talks directly to xAI realtime websocket
- So the safe policy here is: **strict non-env auth for session creation and transcribe**. If a future server-side “continue live voice session” endpoint is added, it should use a dedicated short-lived signed voice-session token bound to a prior authenticated session.

## What was wrong
The routes already checked `locals.sessionUser`, but `locals.sessionUser` was populated by `resolveSessionUser()`, which allowed env fallback via `AG_WEBUI_ALLOW_ENV_AUTH`. In practice, when env auth was enabled, an unauthenticated request could reach these voice routes as the configured operator user.

## Code changes
1. Added an opt-out for env fallback in `ag-webui/src/lib/server/session.ts`:
   - `resolveSessionUser(..., { allowEnvAuth: false })`
   - convenience helper `resolveInteractiveSessionUser(...)`
2. Switched voice routes to the strict helper:
   - `ag-webui/src/routes/api/chat/transcribe/+server.ts`
   - `ag-webui/src/routes/api/chat/live-voice/session/+server.ts`
3. Added regression coverage in `tests/test_ag_webui_api_auth.py` for:
   - unauthenticated rejection even when env auth is enabled
   - trusted proxy success path even when env auth is enabled

## Tests run
- `npm run check` (in `ag-webui`) ✅
- `pytest tests/test_ag_webui_api_auth.py` ✅ `20 passed in 5.01s`

## Notes / residuals
- This fixes the requested voice-route policy gap without touching billing or secret plumbing.
- `AG_WEBUI_ALLOW_ENV_AUTH=true` still exists as a broader product setting and may still permit env-auth on other routes. I left that broader product policy unchanged because the task was specifically about the voice endpoints.
