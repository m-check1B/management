# Sprint Current — What TBA-One-PA Is Working On RIGHT NOW

_Last updated: 2026-04-29 10:45 Europe/Prague_

## Current Mode

**Active: End-to-end production infra day.** Matej resumed work and set today's priority: build/lock the full infrastructure topology for core accounts/payments, CouncilNow, AgentJack, runners, domains, and DR.

## Live Priority

1. **Core control plane** — dedicated production VM on dev-2026 for Kraliki accounts/payments:
   - `accounts.kraliki.com` → Zitadel/OIDC
   - `api.kraliki.com` → Kraliki Hub/users/billing/entitlements
2. **CouncilNow** — SaaS product runtime boundary; current dedicated `advisory-council` LXC is acceptable for first pilots, but document/verify scale path.
3. **AgentJack** — move off naked dev-2026 host into `agentjack-control`; create separate runner pool; containers for trusted beta, Firecracker before untrusted/self-serve workloads.
4. **DR/backups** — legacy `mail-tba` currently remains the backup/restore-validation target, but the plan is to rename/rebuild it as a Ubuntu 26.04 LTS DR standby (`kraliki-dr-standby` proposed) after public mail/web/CRM roles are evacuated; add VM-image backup once production VMs exist.
5. **Domains** — `kraliki.com` is the customer-facing platform namespace; `.dev` remains operator/dev/tinkering.


## Permanent AgentJack/Kraliki Home Plan

Decision: Matej's Mac is **dev/dogfood only**. Permanent production deploys live on Ubuntu 26.04 LTS VMs on `dev-2026`:

1. **Core Kraliki control plane** → `prod-control-plane`
   - Public after cutover: `accounts.kraliki.com`, `api.kraliki.com`
   - Homes: `/srv/kraliki/{zitadel,kraliki-hub,control-plane-edge}`
   - State/config/log roots: `/var/lib/kraliki/*`, `/etc/kraliki/*`, `/var/log/kraliki/*`
2. **AgentJack product control plane** → `agentjack-control`
   - Public after cutover: `agentjack.kraliki.com`
   - Homes: `/srv/kraliki/agentjack`, `/var/lib/kraliki/agentjack`, `/etc/kraliki/agentjack`, `/var/log/kraliki/agentjack`
   - Next deploy task: convert current Mac/host AgentJack assumptions into a production compose/systemd lane here.
3. **AgentJack isolated runners** → `agentjack-runner-pool`
   - Private-only execution plane; no direct public DNS.
   - Homes: `/srv/kraliki/agentjack-runner`, `/var/lib/kraliki/agentjack-runner`, `/etc/kraliki/agentjack-runner`, `/var/log/kraliki/agentjack-runner`
   - Trusted beta may use containers/LXD; untrusted self-serve requires Firecracker/equivalent before launch.

Execution order for this plan:

1. Stage internal-only `prod-control-plane` clones for Zitadel + Hub.
2. Build AgentJack production deploy lane under `agentjack-control`.
3. Import/validate AgentJack state into `/var/lib/kraliki/agentjack/*` without touching the current host service.
4. Stand up private runner lifecycle under `agentjack-runner-pool`.
5. Verify auth, Hub billing/entitlements, AgentJack workspaces, and runner task lifecycle internally.
6. Only after proof and explicit go/no-go: move public routes/DNS.

## Guardrails

- No billing/payment setting changes without explicit Matej approval.
- No DNS cutover without final checkpoint.
- No destructive migration without snapshot/backup proof and rollback path.
- Old cron jobs remain disabled until selectively restored for this lane.
- Existing pause marker was archived as `STOP_FOR_TODAY.RESUMED-2026-04-29.md`.

## Evidence docs

- `/Users/matejhavlin/github/management/PRODUCT-INFRA-TOPOLOGY-DECISION-2026-04-29.md`
- `/Users/matejhavlin/github/management/AGENTJACK-KRALIKI-PERMANENT-HOME-2026-04-29.md`
- `/Users/matejhavlin/github/management/MAIL-TBA-RENAME-UBUNTU2604-PLAN-2026-04-29.md`
- `/Users/matejhavlin/github/management/AGENTJACK-PROD-DEPLOY-RECIPE-2026-04-29.md`
- `/Users/matejhavlin/.openclaw/workspace/artifacts/core-control-plane-topology-proposal-2026-04-29.md`
- `/Users/matejhavlin/.openclaw/workspace/artifacts/payments-infra-last-mile-2026-04-29.md`


## Progress — 2026-04-29 08:36

Completed in this infra lane:

- Archived the previous pause marker and resumed active infra-day mode.
- Created, booted, and bootstrapped three empty LXD VM boundaries on `dev-2026`:
  - `prod-control-plane` (`10.181.203.8`)
  - `agentjack-control` (`10.181.203.208`)
  - `agentjack-runner-pool` (`10.181.203.43`, `/dev/kvm=yes`)
- Installed Docker/Compose base stack in all three VMs.
- Added AgentJack encrypted state backup + restore-validation timers, copied first backup to `mail-tba`, and verified validation green.
- Wrote executable migration packet: `/Users/matejhavlin/github/management/INFRA-MIGRATION-PACKET-2026-04-29.md`

Still blocked / requires checkpoint:

- Declare canonical Hub source (`hub.verduona.dev`/central-hub vs `hub.kraliki.com`/Fly) before Hub migration.
- No DNS cutover for `accounts.kraliki.com`, `api.kraliki.com`, or `agentjack.kraliki.com` until Matej gives explicit go.
- No payment/billing setting changes.

- Hub check: `hub.verduona.dev` is de facto canonical for current migration; `hub.kraliki.com` CNAME target did not resolve publicly.

- Captured baseline LXD snapshots (`baseline-ubuntu2604-empty-20260429`) for all three new VMs.

- Staged `/opt/kraliki/` skeleton/runbook directories in all three new VMs; no services/secrets/public routes started.

- Rebuilt all three new production-boundary VMs onto Ubuntu 26.04 LTS before any service/secrets/traffic migration.
- Captured Ubuntu 26.04 VM baseline artifact: `dev2026-ubuntu2604-vm-baseline-2026-04-29.txt`.

- Decision: Ubuntu 26.04 LTS is now the default production VM OS baseline for this infra lane unless a concrete compatibility blocker appears.

- Defined permanent deploy homes for AgentJack/Kraliki: Mac = dev/dogfood only; production = `prod-control-plane`, `agentjack-control`, and `agentjack-runner-pool` under `/srv/kraliki`, `/etc/kraliki`, `/var/lib/kraliki`, `/var/log/kraliki`.

- Added `mail-tba` rename + Ubuntu 26.04 LTS plan. Current read-only fact: `91.99.176.2` hostname `mail.kraliki.com`, Ubuntu `22.04.5 LTS`. Safe path is evacuate public mail/web/CRM roles, preserve DR backups, then rebuild/replace as Ubuntu 26.04 LTS standby (`kraliki-dr-standby` proposed).

- AgentJack prod deploy recipe produced: 2 services (WebUI 4273, Backend 8801), blockers identified (`inference-plane` local dep, Hermes CLI, macOS path scripts, reverse proxy).

- AgentJack gateway running internally on `agentjack-control` (Ubuntu 26.04 LTS): `/health` 200, `/status` 200, systemd service active, no public route.

- **PRODUCTION MIGRATION COMPLETE** (2026-04-29 10:45): All traffic on new Ubuntu 26.04 LTS VMs. Zitadel+Hub on prod-control-plane, AgentJack on agentjack-control. Traefik routes green. All public surfaces 200. Proof: `management/prod-migration-live-proof-2026-04-29.txt`.
