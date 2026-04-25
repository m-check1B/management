# DR Standby Plan — mail-tba (Hetzner)

_Last updated: 2026-04-25 18:02 CET_

## Objective
Repurpose `mail-tba` as a disaster-recovery standby node for critical data continuity (Hub/auth/users/invoicing), while keeping primary runtime on `dev-2026`.

## Target Reliability
- RPO target: 5–15 minutes
- RTO target: 30–60 minutes

## Scope
- In scope: critical databases and backups for Hub/auth/billing-related systems.
- Out of scope: public website hosting, public app runtime, general-purpose app deployment.

## Architecture
- Primary: `dev-2026`
- DR standby: `mail-tba`
- Mail providers: Resend (transactional/app outbound) + Proton Mail (human mailbox/ops)

## Initial critical data inventory (2026-04-25 snapshot)
- `advisory-zitadel-db` (Postgres) — identity/auth continuity
- `zitadel-db` (Postgres) — identity/auth continuity (legacy/parallel lane; confirm canonical one)
- `kraliki-postgres` (Postgres) — likely Hub/core app data (confirm schema ownership)
- `twenty-db` (Postgres) — CRM/business records (confirm invoicing dependency)
- `kiki-db` (Postgres) — app-specific data (lower priority unless linked to critical billing/auth flows)

Note: Phase A must confirm which DBs are canonical for users/auth/invoicing before DR replication policies are finalized.

## Phased Execution
1. **Phase A — Data classification**
   - List critical DBs/tables for users/auth/invoicing.
   - Define retention and encryption requirements.
2. **Phase B — Backup hardening**
   - Implement scheduled encrypted DB backups to `mail-tba`.
   - Add backup integrity checks (checksum + restore-validate).
3. **Phase C — Standby readiness**
   - Prepare restore automation on `mail-tba`.
   - Optional: add warm-standby replication where practical.
4. **Phase D — Failover drill**
   - Run controlled failover simulation.
   - Measure actual RPO/RTO and close gaps.

## Acceptance Criteria
- Daily backup integrity report is green.
- Monthly restore drill succeeds.
- Documented failover runbook exists and is tested.
- No critical public surface depends on `mail-tba` in normal operation.
