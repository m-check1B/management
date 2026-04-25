# TBA-One-PA ↔ Axis (dev-2026) Coordination Policy

_Last updated: 2026-04-25_

## 1) Role split (locked)

### TBA-One-PA on Mac (control plane)
- Owner of founder-facing coordination.
- Owns prioritization, sequencing, acceptance criteria, and final go/no-go calls.
- Converts founder direction into executable work packets.
- Owns cross-project context (AgentJack first, then verticals).

### Axis Agent on dev-2026 (execution plane)
- Owner of continuous execution/autodev on server lane.
- Runs long jobs, runtime checks, infra surgery, and implementation loops.
- Produces evidence artifacts (logs, diffs, test outputs, health probes).
- Does not redefine strategy; executes against declared priorities.

## 2) Source-of-truth hierarchy
1. Live founder instruction in active chat.
2. `/Users/matejhavlin/github/management/SPRINT-CURRENT.md`
3. Canonical infra docs:
   - `/Users/matejhavlin/github/infra/MAIL-VM-MIGRATION-PLAN.md`
   - `/Users/matejhavlin/github/infra/PRODUCTION-ARCHITECTURE.md`
4. Runbooks/trackers (deployment, DR, lane maps).

If signals conflict, higher tier wins.

## 3) Work routing rules
- Revenue/auth/billing/runtime regressions = P0 lane, immediate.
- TBA packages work; Axis executes.
- Linear is the canonical execution queue.
- Linear operations use **Python GraphQL path first**; MCP fallback only.
- Keep one active top priority lane at a time; avoid horizontal spread.

## 4) Handoff contract (required)
Every delegated item must include:
- Objective (single sentence)
- Scope boundaries (in/out)
- Acceptance checks (exact commands or API checks)
- Evidence format required
- Rollback or containment rule

No handoff without verifiable acceptance criteria.

## 5) Update protocol
Axis reports only on:
- completion,
- blocker needing decision,
- time-sensitive risk,
- failed verification requiring intervention.

No status theater. Evidence-first updates only.

## 6) Deployment/ops standard
- Build → Deploy → Verify is mandatory logging for infra/app changes.
- Websites lane: Cloudflare.
- Non-mail runtimes/apps: dev-2026.
- `mail-tba`: DR standby for critical data continuity.

## 7) Safety and authority
- No payments/billing/spending changes without Matej.
- Destructive/irreversible ops require explicit approval.
- External commitments/public comms require founder alignment.

## 8) Escalation
- Urgent/time-sensitive blocker: voice escalation path.
- Async non-urgent attention: Telegram ping path.

## 9) Quality gate
A task is not done until evidence is attached:
- passing tests/checks OR
- explicit blocker with root cause + next required decision.

## 10) Anti-overload principle
Prefer specialized repeatable lanes (solo-purpose agents/work packets) over one broad-context agent doing everything.
This policy intentionally optimizes depth and repeatability over context sprawl.

## 11) Pi-style specialist adoption (active)
- Operational detail is defined in:
  - `/Users/matejhavlin/github/management/PI-SPECIALIST-AGENT-ADOPTION.md`
- That document defines active specialist lanes, packet contract, verification chain, and 7-day pilot metrics.
- Usage focus is repetitive, long-horizon, narrow tasks where a lane compounds quality over time.
- Where this file and the Pi-style doc overlap, use the stricter rule.
