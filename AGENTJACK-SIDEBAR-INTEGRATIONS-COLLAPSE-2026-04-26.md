# AgentJack Sidebar Integrations Collapse Fix — 2026-04-26

## Symptom

`Integrations` sidebar group did not collapse on `/plugins`.

## Root Cause

The previous implementation forced the group containing the active route to remain open. On `/plugins`, clicking `Integrations` persisted `{"integrations": true}`, but rendering still evaluated the group as open because it had the active item.

## Fix

- `ag-webui/src/lib/components/layout/Sidebar.svelte`
  - Removed active-route gate from sidebar group collapse state.
  - Group collapse now follows the persisted group toggle state directly when the global sidebar is expanded.
- `ag-webui/tests/sidebar-group-collapse.test.mjs`
  - Added a regression test that fails if group collapse is gated by active route logic again.

## Git

- AgentJack commit: `9d505fc Allow active sidebar groups to collapse`

## Verification

Local:

```text
cd ag-webui && npm run check
svelte-check found 0 errors and 0 warnings

cd ag-webui && node tests/sidebar-group-collapse.test.mjs
sidebar-group-collapse: ok

cd ag-webui && npm run build
passed

git diff --check -- ag-webui/src/lib/components/layout/Sidebar.svelte ag-webui/tests/sidebar-group-collapse.test.mjs
passed
```

Dev-2026 deploy:

```text
git pull --ff-only
Fast-forward to 9d505fc

cd ag-webui && npm run check
svelte-check found 0 errors and 0 warnings

cd ag-webui && node tests/sidebar-group-collapse.test.mjs
sidebar-group-collapse: ok

cd ag-webui && npm run build
passed

systemctl --user restart agentjack-ag-webui.service
active

curl http://127.0.0.1:4313/health
{"ok":true,"service":"agentjack-ag-webui",...}
```

Public checks:

```text
./scripts/check-public-ag-webui.sh
public ag-webui check passed.

./scripts/check-dev2026-deploy.sh
deployment check passed with 3 warning(s).
```

Warnings are existing/non-blocking:

- reverse tunnel inactive, skipped
- Telegram inbound continuity still pending token confirmation

Browser proof on public dev:

- URL: `https://agentjack-dev.verduona.dev/plugins?collapse-verify=9d505fc`
- Click `Integrations`:
  - `aria-expanded=false`
  - label: `Expand Integrations navigation group`
  - localStorage: `{"integrations":true}`
  - Integrations links hidden
- Click `Integrations` again:
  - `aria-expanded=true`
  - localStorage: `{"integrations":false}`
  - Integrations links visible

## Status

Done.
