# OpenClaw vs AgentJack — Side-by-Side Audit (Current Iteration)

Date: 2026-04-26 06:35 UTC (08:35 CEST)
Scope: Live runtime + control-plane readiness on `dev-2026`

## Executive Summary

- **OpenClaw is operational but currently has critical security posture gaps** (gateway auth/sandboxing) and state-integrity warnings.
- **AgentJack runtime lane is now correctly isolated on dedicated gateway and Telegram polling is healthy**, but **private webchat readiness is blocked** and integrations are still mostly in planned/attention states.
- Highest-value near-term move: **close OpenClaw security criticals + fix AgentJack private webchat readyz**; then lock diagnostics consistency and continue integration extraction.

## Evidence (live checks run)

- `openclaw status`
- `openclaw doctor`
- `openclaw security audit --json`
- `systemctl --user list-units --type=service --all | rg 'openclaw-gateway|agentjack-gateway|hermes-gateway-axis'`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli runtime-status --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli telegram-status --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli telegram-conflicts --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli integrations-overview --pretty`
- `PYTHONPATH=src .venv/bin/python -m agentjack.cli system-doctor --pretty`
- `curl -sS https://agentjack-dev.verduona.dev/readyz`
- `curl -sS http://127.0.0.1:4173/readyz`

---

## Side-by-side audit matrix

| Area | OpenClaw (current) | AgentJack (current) | Gap / Risk | Priority |
|---|---|---|---|---|
| Runtime service ownership | `openclaw-gateway.service` active | `agentjack-gateway.service` active; `hermes-gateway-axis.service` disabled/inactive | Ownership is now correct, but both stacks run concurrently and need strict lane separation discipline | Medium |
| Telegram lane health | Telegram channel reports OK (`@TbaTwoBot`) | `telegram-status`: ready, live delivery ready, no active contenders | Core conflict issue resolved; remaining continuity proof still human-token gated | Low |
| Telegram continuity proof | Not enforcing comparable per-test token proof in doctor output | `telegram-doctor`: `attention` (awaiting reply token `AJ-472513`) despite healthy polling | Diagnostic strictness mismatch causes operational noise / false urgency | Medium |
| Security posture | `openclaw security audit`: **2 critical**, 2 warn (`gateway auth missing`, `HTTP APIs without auth`, `small models unsandboxed`) | No equivalent critical security finding in current AgentJack doctor output; runtime lane ready | OpenClaw control plane is currently the larger security exposure | **High** |
| Web surface readiness | Control UI reachable via local gateway | Public AG WebUI readyz OK, but private/local `http://127.0.0.1:4173/readyz` unreachable; `system-doctor` blocked on private webchat | AgentJack private operator surface degraded (public looks healthy, private path not) | **High** |
| Integrations maturity | Rich runtime/tooling ecosystem already operational | Integrations plane totals: healthy=2, attention=4, planned=13 | AgentJack extraction maturity gap vs OpenClaw breadth | **High** |
| State integrity / housekeeping | Doctor reports multi-state-dir + orphan/missing transcript warnings | Runtime/telegram status coherent; some historical artifacts still reference Hermes profile paths in delivery history | OpenClaw state hygiene weaker; AgentJack has residual provenance drift in historical records | Medium |
| Fleet/heartbeat ops | Main heartbeat active; other agents disabled | Dedicated gateway running; integration modules still mostly planned | Operational asymmetry: OpenClaw mature ops, AgentJack still building modules | Medium |
| Repo hygiene (AgentJack) | N/A | Working tree contains many modified/deleted files (including large `webchat-legacy` generated/deleted artifacts) | High merge/release risk and review friction | **High** |

---

## Top gap list (prioritized)

### P0
1. **OpenClaw security criticals unresolved**
   - Gateway auth and HTTP surface exposure + sandboxing policy for small models.
2. **AgentJack private webchat path unhealthy**
   - `system-doctor` blocked because local/private readyz does not report healthy release.

### P1
3. **AgentJack integration extraction lag**
   - Only GitHub + Linear healthy; many production-relevant surfaces still planned/attention.
4. **Diagnostics consistency mismatch (continuity proof vs runtime health)**
   - Telegram lane healthy but doctor remains attention until manual token reply.

### P2
5. **State hygiene / provenance cleanup**
   - OpenClaw orphan/missing transcript cleanup.
   - AgentJack historical delivery records still mixing old Hermes profile-home lineage.

---

## Recommended execution plan (7-day)

1. **Security hardening lane (OpenClaw)**
   - Enable gateway auth mode (token/password), lock trusted proxies, enforce sandbox/web-tool policy for small models.
   - Re-run `openclaw security audit --deep` until critical=0.

2. **AgentJack private surface recovery**
   - Restore local/private readyz health on `127.0.0.1:4173` (or retire path explicitly if no longer canonical).
   - Make `system-doctor` green in private mode.

3. **Integration extraction sprint (AgentJack)**
   - Promote 2 attention surfaces to healthy (Slack + Obsidian or Notion) based on near-term business need.
   - Keep write-path checks behind explicit readiness gates.

4. **Diagnostics normalization**
   - Split “transport healthy” from “human continuity proof pending” to avoid masking runtime green states.

5. **Repo hygiene sweep (AgentJack)**
   - Isolate intentional changes from generated/deletion noise; create clean, reviewable commits.

---

## Suggested Linear deliverables

- **Epic:** OpenClaw ↔ AgentJack convergence audit closure
- **Issue A (P0):** OpenClaw security criticals to zero
- **Issue B (P0):** AgentJack private webchat readyz recovery
- **Issue C (P1):** AgentJack integrations extraction (next 2 surfaces to healthy)
- **Issue D (P1):** Telegram doctor signal split (health vs proof)
- **Issue E (P2):** State/provenance cleanup (OpenClaw + AgentJack)

---

## Current verdict

- **OpenClaw:** mature operations, but currently carrying the most serious security debt.
- **AgentJack:** runtime isolation and Telegram stability are now materially improved; biggest gaps are private web surface reliability and integration completeness.
