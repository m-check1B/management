# AgentJack Autonomous VM Lab — Proposal

_Recorded: 2026-04-28 17:12 CEST_

## Question

Would it be workable to put OpenClaw, Hermes, and current-state AgentJack into an isolated VM, provide tokens, and let them self-develop?

## Execution status

**HOLD — do not run this yet.**

Matej explicitly said on 2026-04-28 17:15 CEST: **do not do it with agents on VM, not yet.**

This document is retained as a design/proposal only. It is not authorization to create VM agents, install autonomous workers, provide tokens, start self-development loops, or schedule any cron/heartbeat around this idea.

## Short answer

**Technically workable later as a sandboxed self-development lab. Not approved yet, and not safe as an unconstrained production organism.**

The right model is a contained autonomous dev cell:

- isolated VM / microVM
- scoped credentials
- branch/PR-only write path
- hard test/build/security gates
- external watchdog with kill/reset authority
- no production billing/DNS/payment/admin tokens

## Why this is worth trying

AgentJack is now clean and gated locally after the 2026-04-28 yolo pass:

- `agentjack` `main` at `ca0a98a`
- repo clean
- full Python suite passed (`582 passed, 6 skipped, 16 subtests passed`)
- AG WebUI check/strict JS/build passed
- product vertical check/build/smoke gates passed

That makes it a good candidate for a reproducible VM experiment. The current state is stable enough to serve as a baseline snapshot.

## Non-negotiable boundaries

Do **not** give the autonomous VM broad production power.

Forbidden by default:

- production Stripe/billing/payment tokens
- DNS write tokens
- Cloudflare zone-edit tokens
- production database admin credentials
- Telegram/Signal/WhatsApp live user-facing bot tokens shared with OpenClaw/Hermes
- direct push to `main`
- live deploy authority
- secret-store mutation authority

Prior issue to remember: OpenClaw/Hermes/AgentJack token ownership drift already caused Telegram polling/identity conflicts. In the VM lab, each runtime must use isolated staging tokens and separate bot/channel ownership.

## Recommended architecture

### 1. Baseline VM snapshot

- clean OS image
- OpenClaw installed
- Hermes available only if explicitly needed for compatibility tests
- AgentJack checkout at a known commit
- reproducible package/cache setup
- reset-to-snapshot path after failed cycles

### 2. Scoped token bundle

Use least privilege:

- GitHub token limited to AgentJack repo and PR branches
- model/API keys with budget/rate caps
- staging bot/channel tokens only
- read-only observability tokens where possible
- no billing/DNS/payment/admin secrets

### 3. Autonomy contract

The VM may:

- inspect repo state
- select from a bounded task queue
- create branches
- edit code/docs/tests
- run checks/builds/smokes
- open PRs
- write evidence reports

The VM may not:

- merge PRs
- deploy live
- change production secrets
- send external messages
- create spending/billing changes
- disable safeguards

### 4. Hard gates

Every self-development cycle must pass:

- `git status` sanity before start
- secret scan before commit/PR
- relevant lint/check/build/test gates
- diff-size threshold
- no critical-file deletion without explicit review
- no token/log leakage in artifacts
- PR summary with evidence

### 5. External watchdog

A supervisor outside the VM should track:

- runtime/cost limits
- repeated failed cycles
- excessive file churn
- network egress anomalies
- secret-scan failures
- loops with no useful PR output

The watchdog can pause, kill, or reset the VM.

## First experiment

Run a 24–48h staging-only trial.

Success criteria:

1. Produces 3 small useful PRs.
2. Every PR has tests/build proof.
3. No secret leakage.
4. No broad noisy rewrites.
5. No live deploy or external side effects.
6. Watchdog cost/runtime limits stay under threshold.

Failure criteria:

- repeated huge diffs
- failing gates after multiple cycles
- stale-task resurrection
- self-modifying cron/heartbeat loops
- any secret exposure
- any attempt to access production billing/DNS/payment surfaces

## Recommended first task queue

Start with bounded tasks that are useful and low-risk:

1. Add/update smoke tests for AgentJack app-boundary verticals.
2. Improve AG WebUI readiness/status truthfulness.
3. Tighten docs around runtime contracts.
4. Add small regression tests for recent failures.
5. Reduce noisy cron/autodev behavior in the isolated lab only.

Avoid first:

- production Telegram delivery
- billing flows
- public deploys
- DNS/cert automation
- broad UI rewrites
- self-modifying memory/identity/cron systems

## Verdict

Proceed, but as a **contained autonomous development cell** with scoped staging credentials and PR-only output.

If it can produce clean, useful PRs for two days without churn or leaks, scale privileges carefully. If it creates noise, keep it as an experiment and do not connect it to production.
