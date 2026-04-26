# KRA-3680 — OpenClaw vs AgentJack Side-by-Side Gap Report

Date: 2026-04-26
Baseline issue snapshot: 06:35 UTC / 08:35 CEST
Fresh verification: ~07:45 UTC / ~09:45 CEST
Scope: OpenClaw operator runtime on Matej Mac + AgentJack current product/runtime lane, especially dev-2026 AG WebUI.

## Executive Summary

The original KRA-3680 audit was directionally useful, but several claims are now stale after today’s AgentJack and website work.

- **OpenClaw remains operational and mature, but still carries the clearest hard security gap:** current `openclaw security audit --json` reports **1 critical / 3 warn / 1 info**. The previous “gateway auth missing / HTTP APIs without auth” criticals are no longer present in the current local audit; the remaining critical is **small-model fallback policy with sandbox/web-tool exposure**.
- **AgentJack has advanced materially today:** AG WebUI public/dev deployment is green, chat runtime model/thinking controls are shipped, Human Queue is now actionable, voice routes are guarded, and the public dev surface is verified.
- **The old AgentJack `127.0.0.1:4173` private readyz blocker should be downgraded to topology/runbook drift, not treated as a product P0.** The canonical dev-2026 direct AG WebUI service is `agentjack-ag-webui.service` on port `4313`, and the public route `https://agentjack-dev.verduona.dev` passes deployment checks.
- **AgentJack’s real remaining product gaps are integrations maturity, repo/release hygiene, SaaS/containerization foundations, and diagnostics normalization.**
- **Telegram transport is healthy; continuity proof signaling is noisy.** `telegram-status` is ready and no same-bot conflict is suspected, but public `readyz` still reports inbound continuity not verified for pending token `AJ-472513`.

Bottom line: **reframe KRA-3680 from “AgentJack private readiness blocked” to “OpenClaw security P0 remains; AgentJack product surface is now green enough to keep building, with gaps in integrations, SaaS foundations, repo hygiene, and diagnostic truthfulness.”**

---

## Fresh Evidence Run

### OpenClaw

Commands run:

- `openclaw status`
- `openclaw doctor`
- `openclaw security audit --json`

Observed:

- Gateway reachable on local loopback: `ws://127.0.0.1:18800`.
- Gateway service active via LaunchAgent.
- Telegram channel: `ON / OK`.
- Memory: vector + FTS ready.
- Security audit summary: **1 critical / 3 warn / 1 info**.
- Remaining critical:
  - `models.small_params`: small fallback models (`groq/llama-3.3-70b-versatile`, `groq/llama-3.1-8b-instant`) are unsafe with sandbox off and web/browser tools available.
- Warnings:
  - reverse proxy headers not trusted,
  - phantom `plugins.allow` entries,
  - extension plugin tools reachable under permissive tool policy.
- Doctor state hygiene warning:
  - `108` orphan transcript files in `~/.openclaw/agents/main/sessions`.
- Doctor still recommends single gateway per machine for most setups, while several gateway-like services exist.

### AgentJack

Commands run:

- `PYTHONPATH=src .venv/bin/python -m agentjack.cli runtime-status --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli telegram-status --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli telegram-conflicts --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli integrations-overview --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli system-doctor --pretty`
- `./scripts/check-public-ag-webui.sh`
- `./scripts/check-dev2026-deploy.sh`
- dev-2026 direct service checks over SSH

Observed:

- Runtime:
  - `gateway_declared_state`: `running`
  - `gateway_status_ok`: `true`
  - `execution_ready`: `true`
  - `fallback_ready`: `true`
  - `readiness_status`: `ready`
  - usable model checks: `21 / 29`
  - `direct_runtime_ready`: `false` due missing optional provider keys (`KIMI_API_KEY`, `XIAOMI_MIMO_API_KEY`, `GROQ_API_KEY`) in AgentJack runtime env.
- Telegram:
  - `readiness_status`: `ready`
  - `delivery_ready`: `true`
  - `live_delivery_ready`: `true`
  - `operator_ready`: `true`
  - `live_gateway_token_aligned`: `true`
  - `suspected_conflict_count`: `0`
  - two active gateway processes are expected/different-token lanes: `com.agentjack.gateway` and `ai.openclaw.gateway`.
- Integrations plane:
  - modules: `5`
  - healthy: `1` (`GitHub`)
  - attention: `4` (`Slack`, `Notion`, `Obsidian`, `Linear`)
  - blocked: `0`
  - planned: `13`
  - Important nuance: Linear works for this operator via local secret path, but AgentJack’s integrations runtime does not currently see `LINEAR_API_KEY`, so the AgentJack Linear surface remains `attention`.
- AG WebUI / dev-2026:
  - `agentjack-ag-webui.service`: active/running on dev-2026.
  - dev-2026 local direct health: `http://127.0.0.1:4313/health` → `ok: true`.
  - public route: `https://agentjack-dev.verduona.dev/readyz` → `ok: true`, runtime execution ready, live Telegram delivery ready.
  - `./scripts/check-public-ag-webui.sh` passed.
  - `./scripts/check-dev2026-deploy.sh` passed with 3 warnings:
    - reverse tunnel inactive; skipped because direct service is canonical,
    - Telegram delivery live but inbound continuity not verified,
    - pending token guidance: `AJ-472513`.
- Local/private diagnostic drift:
  - `agentjack.cli system-doctor` still checks `http://127.0.0.1:4173` and reports `doctor_status: blocked` because that readyz does not respond.
  - This conflicts with the now-canonical dev-2026 direct service (`4313`) and public deployment checks. Treat as runbook/diagnostic drift unless the local Mac private surface is intentionally restored as a product requirement.
- Repo hygiene:
  - core AgentJack code tree is clean after excluding `agentjack-autodevelopment`.
  - `agentjack-autodevelopment` has `87` modified/deleted/untracked generated or steering files. This is still a release/review hygiene risk, but not evidence of dirty core runtime code.

---

## Side-by-Side Matrix — Refreshed

| Area | OpenClaw current | AgentJack current | Gap / Risk | Priority |
|---|---|---|---|---|
| Runtime ownership | Local OpenClaw gateway active and reachable; mature operator runtime | Dedicated AgentJack gateway active; dev-2026 AG WebUI service active | Lanes are separated enough operationally, but same-machine gateway-like service noise still exists | Medium |
| Security posture | **1 critical / 3 warn**; remaining critical is small-model sandbox/web policy | No equivalent current critical surfaced by AgentJack checks | OpenClaw still has the hard P0 security gap; baseline “gateway auth missing” appears stale | **High** |
| Web/operator surface | Control UI reachable locally | Public AG WebUI checks pass; dev-2026 direct service healthy on `4313`; Human Queue and chat runtime controls shipped | Old `4173` private readyz is diagnostic/runbook drift unless still required | Medium |
| Telegram transport | OpenClaw Telegram OK | AgentJack Telegram ready, live delivery ready, no same-bot conflict suspected | Transport healthy on both; proof-state/readyz wording is noisy | Medium |
| Telegram continuity proof | Pairing/group-policy warnings remain normal setup debt | Public readyz still says inbound continuity pending token `AJ-472513` | Split “delivery transport healthy” from “human proof token pending” in diagnostics | Medium |
| Integrations breadth | Rich OpenClaw tool/channel ecosystem already usable | AgentJack integrations: `1 healthy`, `4 attention`, `13 planned` | AgentJack product extraction maturity still lags OpenClaw breadth | **High** |
| State hygiene | 108 orphan transcripts; gateway-like service warnings | Historical/autodev artifacts and delivery records still noisy | Cleanup needed, but not the top revenue/product blocker | Medium |
| Repo hygiene | N/A | Core code clean, but `agentjack-autodevelopment` has 87 dirty/generated changes | Review/release friction; isolate or ignore generated state intentionally | **High** |
| SaaS/container foundation | N/A / workbench product, not SaaS surface | Product still needs stronger container/bootstrap/instance-root story | Required for AgentJack B2B productization | **High** |

---

## Claims From Baseline To Downgrade / Correct

1. **“OpenClaw has 2 critical security findings including gateway auth missing.”**
   - Current audit shows **1 critical**, not 2.
   - Gateway is local loopback and reports auth-token access.
   - The remaining critical is small-model sandbox/web-tool policy.

2. **“AgentJack private AGChat readiness is a P0 product blocker.”**
   - Current public/dev deployment checks pass.
   - dev-2026 canonical direct service is `4313`, not `4173`.
   - Keep a diagnostic/runbook issue for stale `4173` assumptions.

3. **“AgentJack Telegram continuity is a transport blocker.”**
   - Transport is ready and live delivery is ready.
   - Remaining issue is proof-state persistence/readyz wording around manual token confirmation.

4. **“AgentJack operator surface is broadly not ready.”**
   - Too broad after today’s work.
   - Chat runtime controls, voice guards, Human Queue task execution, and public AG WebUI deployment are now proven.

5. **“CouncilNow / sites are still part of this blocker.”**
   - CouncilNow quality/pricing/dummy E2E blockers are closed.
   - Kiki parity and marketing visual restoration are deployed.
   - Resend sender verification remains pending, but that belongs to the CouncilNow outbound-email lane, not this OpenClaw vs AgentJack runtime audit.

---

## Current Prioritized Gap List

### P0 / High

1. **OpenClaw security critical to zero**
   - Decide how to handle small-model fallbacks:
     - remove/disable unsafe small fallbacks from default web-capable lanes, or
     - enforce sandbox + deny web/browser tools when small models can be selected.
   - Re-run `openclaw security audit --json` until `critical=0`.

2. **AgentJack integrations extraction maturity**
   - Promote next internal-business-critical surfaces from `attention/planned` to `healthy`.
   - Recommended next surfaces:
     - Linear: expose existing internal secret safely to AgentJack integrations runtime or configure dedicated env.
     - Obsidian/notes: align with the real workspace memory/vault story.
   - Slack/Notion can wait unless immediate business workflows need them.

3. **AgentJack SaaS/containerization foundation**
   - Define tenant/instance root, container bootstrap, image/runtime boundary, and upgrade path.
   - This is a productization gap, not a live dev-webui blocker.

4. **AgentJack repo/release hygiene**
   - Contain or ignore `agentjack-autodevelopment` generated state intentionally.
   - Keep core product commits reviewable and separate from generated memory/log churn.

### P1 / Medium

5. **Diagnostics normalization**
   - Replace stale `4173` private readyz assumption with canonical dev-2026 service topology (`4313`) or explicitly restore local private service if desired.
   - Split readiness fields:
     - runtime execution healthy,
     - Telegram delivery healthy,
     - inbound proof pending.

6. **OpenClaw state hygiene**
   - Archive orphan transcripts after confirming no active session depends on them.
   - Review gateway-like LaunchAgents; do not remove anything needed for rescue/tunnel workflows without explicit approval.

---

## Recommended Next Linear Split

- **KRA-A — OpenClaw security critical to zero**
  - Scope: small-model fallback policy, sandbox/web-tool gating, plugin allowlist cleanup.
- **KRA-B — AgentJack integration plane: Linear + Obsidian to healthy**
  - Scope: credentials/runtime env, read-only checks first, write-readiness gates second.
- **KRA-C — AgentJack AG WebUI diagnostic topology cleanup**
  - Scope: remove/replace stale `4173` assumptions; align system-doctor with canonical dev-2026 `4313` + public route.
- **KRA-D — AgentJack SaaS instance/container bootstrap spec**
  - Scope: tenant instance root, image/runtime boundary, per-customer deployment model.
- **KRA-E — Autodev state hygiene policy**
  - Scope: decide tracked vs ignored generated state, avoid dirty-tree noise in product release work.

---

## Current Verdict

- **OpenClaw:** still the more mature and broad operator platform, but the current live audit says it still has the highest-severity security gap.
- **AgentJack:** no longer blocked on the public/dev operator surface; runtime isolation, Telegram delivery, chat controls, Human Queue, and public AG WebUI are materially improved. The remaining gaps are productization depth: integrations, SaaS/container foundations, diagnostic truthfulness, and repo hygiene.
