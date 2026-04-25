# Pi Pilot Packets — 2026-04-25

## Packet 1 — KRA-3663 (sqlite portability)
- Lane: `coder`
- Objective: Remove hardcoded sqlite paths and make state path resolution portable.
- Acceptance:
  1. No hardcoded sqlite paths in target files.
  2. Target pytest subset no longer fails with `unable to open database file`.
  3. Config/docs updated for runtime path contract.
- Evidence: diff + pytest output.

## Packet 2 — KRA-3663 verification
- Lane: `codex-reviewer`
- Objective: Attempt to break Packet 1 with adversarial path/env permutations.
- Acceptance:
  1. Repro attempts documented.
  2. Pass/fail verdict with concrete command evidence.
- Evidence: failing repro or explicit pass proof.

## Packet 3 — KRA-3664 (doctor alignment)
- Lane: `coder`
- Objective: Align telegram-doctor + system-doctor to a single truth contract.
- Acceptance:
  1. Shared status mapping logic.
  2. Contradictory outputs eliminated in same env.
- Evidence: paired command outputs + tests.

## Packet 4 — KRA-3665 (CI gates)
- Lane: `coder`
- Objective: Replace stale CI with real merge-blocking gates.
- Acceptance:
  1. Core failures produce non-zero CI result.
  2. Required checks documented and verifiable.
- Evidence: failing run proof + passing run proof.

## Packet 5 — KRA-3666 (voice routes)
- Lane: `kimi-frontend` (with backend patch if required via `coder`)
- Objective: Restore transcribe + live-voice session API behavior.
- Acceptance:
  1. Both endpoints return expected status/body for success and error paths.
  2. Regression test coverage added.
- Evidence: request/response proofs + tests.
