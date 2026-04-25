# Pi-Style Specialist Agent Adoption (Mac ↔ dev-2026)

_Last updated: 2026-04-25 20:53 CET_

## Decision
Adopt the Pi-style operating model now: **small solo-purpose specialist lanes**, orchestrated by TBA-One-PA, instead of broad horizontal context in one agent.

## Why
- Better repeatability
- Less context thrash
- Faster verification loops
- Cleaner failure isolation

## Usage boundary (founder lock)
Use this model primarily for **repetitive, long-horizon, narrow tasks** where a specialist lane can improve over time through self-improving loops.

Do **not** over-apply specialist packetization to one-off tiny tasks unless they are high-risk.

## Topology

### Control plane (Mac)
- **TBA-One-PA** owns prioritization, decomposition, acceptance gates, and founder-facing reporting.

### Execution plane (dev-2026)
- **Axis/OpenClaw** runs continuous implementation/verification loops under specialist role packets.

## Specialist lanes (active)
- `coder` → implementation only
- `codex-reviewer` → adversarial QA / beta-test / regression verification
- `kimi-frontend` → frontend/UI execution
- `claude-infra` → infra/runtime/deploy reliability
- `grok-researcher` → external signal/research
- `claude-copy` → copy/messaging artifacts

## Non-negotiable lane rule
A single run has **one lane purpose**. No mixed-role prompts.

Bad: “Implement + review + deploy + write copy.”
Good: one packet per lane, chained by TBA-One-PA.

## Work packet contract (required)
Every delegated packet must contain:
1. Objective (single sentence)
2. Scope in/out
3. Acceptance checks (exact commands/APIs)
4. Evidence required (logs/diff/tests/screenshots)
5. Blocker escalation condition

Template file:
- `/Users/matejhavlin/github/management/PI-WORK-PACKET-TEMPLATE.md`

## Verification chain
1. Implementer lane produces candidate + evidence.
2. Reviewer lane attempts to break it.
3. If break found: send back as concrete repro, not opinion.
4. If pass: mark ready-for-merge/deploy gate.

## Pilot scope (start now)
Apply Pi-style splitting to current AgentJack queue:
- KRA-3663 (sqlite portability)
- KRA-3664 (doctor alignment)
- KRA-3665 (real CI gates)
- KRA-3666 (voice route regressions)

Pilot packet set:
- `/Users/matejhavlin/github/management/PI-PILOT-PACKETS-2026-04-25.md`

## Pilot metrics (7 days)
Track per issue:
- First-pass acceptance rate
- Rework loops count
- Time-to-evidence-complete
- Token/cost per completed issue
- Escaped regressions after "done"

## Anti-overload guard
If a packet needs more than one specialist role to finish, split it immediately.
No agent should carry broad multi-domain context by default.

## Status
This policy is active immediately for Mac↔dev-2026 coordination and supersedes ad-hoc mixed-role delegation.
