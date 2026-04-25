# DR Standby Plan — mail-tba (Hetzner)

_Last updated: 2026-04-25 22:48 CET_

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

## Phase A decision (closed 2026-04-25)

### Canonical data ownership

| Scope | Canonical runtime/store | Criticality | Decision |
|---|---|---|---|
| Platform identity/auth (`identity.verduona.dev`) | `zitadel-db` (Postgres `zitadel`) on `dev-2026` | Tier 0 | **Canonical** identity store for global auth continuity |
| CouncilNow app + billing + portal state | `council-db` (LXD `advisory-council`, Postgres DBs `advisory_council` + `advisory_council_hub`) | Tier 0 | **Canonical** revenue-critical application store |
| CRM revenue ops | `twenty-db` (Postgres `default`) on `dev-2026` | Tier 1 | Keep in DR scope (customer + pipeline continuity) |
| Legacy/parallel identity lane | `advisory-zitadel-db` | Tier 2 | **Non-canonical** for primary auth; backup daily only until retired/repurposed |
| Legacy core app lane | `kraliki-postgres` | Tier 2 | Not in current critical auth/billing path; backup daily only |
| Kiki app lane | `kiki-db` | Tier 3 | Low priority; daily backup optional (best-effort) |

### Retention policy

- **Tier 0 (identity + CouncilNow billing path):**
  - every 15 minutes logical backup
  - daily encrypted full snapshot retained 35 days
  - weekly snapshot retained 12 weeks
  - monthly snapshot retained 12 months
- **Tier 1 (CRM):**
  - hourly logical backup
  - daily encrypted full snapshot retained 21 days
  - weekly snapshot retained 8 weeks
- **Tier 2/Tier 3 (legacy/non-critical):**
  - daily encrypted snapshot retained 14 days

### Encryption + integrity policy

- Backups are encrypted before leaving `dev-2026` (age/GPG acceptable; one standard only in implementation).
- Transfer to `mail-tba` happens over SSH only.
- Every backup artifact includes SHA-256 manifest.
- Restore validation must verify checksum **before** restore and run DB health query (`SELECT 1`) **after** restore.

### Replication/backup topology policy

- Primary runtime remains `dev-2026`; `mail-tba` is standby target only.
- Backup path: `dev-2026 -> mail-tba` (pull or push, but single canonical lane, no split-brain).
- No production reads/writes are served from `mail-tba` during normal operation.
- Optional warm-standby replication is allowed only for Tier 0 stores after checksum+restore automation is green.

## Phased Execution
1. **Phase A — Data classification** ✅ **closed 2026-04-25**
   - Critical canonical DB ownership decided (see Phase A decision section).
   - Retention/encryption/replication policy defined.
2. **Phase B — Backup hardening** ✅ **implemented 2026-04-25 (v1)**
   - Scheduled encrypted DB backups to `mail-tba` are live on `dev-2026` (`systemd --user`):
     - `dr-postgres-backup-tier0.timer` (every 15 min)
     - `dr-postgres-backup-tier1.timer` (hourly)
     - `dr-postgres-backup-tier23.timer` (daily 03:20)
   - Backup script path: `/home/adminmatej/.local/bin/dr-postgres-backup.sh`
   - Encrypted artifact location:
     - source: `/home/adminmatej/backups/dr-postgres/<tier>/.../*.tar.gpg`
     - standby copy: `/home/adminmatej/dr-backups/postgres/<tier>/.../*.tar.gpg` (on `mail-tba`)
   - Integrity gates implemented:
     - encrypted bundle checksum: `*.tar.gpg.sha256`
     - plaintext dump checksum manifest inside bundle: `SHA256SUMS`
   - Restore-validate gate is live:
     - `dr-postgres-restore-validate.timer` (daily 04:10)
     - script: `/home/adminmatej/.local/bin/dr-postgres-restore-validate.sh`
     - validation: checksum verify -> decrypt -> restore into ephemeral Postgres -> `SELECT 1` per DB
   - Validation proof captured in logs:
     - `/home/adminmatej/github/logs/dr-postgres-backup.log`
     - `/home/adminmatej/github/logs/dr-postgres-restore-validate.log`
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
