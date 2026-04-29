# Infra Migration Packet — Kraliki Production Topology

_Date: 2026-04-29 08:36 Europe/Prague_  
_Status: executable prep packet; no DNS/payment cutover executed_

## Executive state

We moved from architecture-only into reversible production boundaries.

Completed today:

- Replaced earlier empty Ubuntu 24.04 bootstrap VMs with 26.04 LTS before any service/secrets/traffic migration.
- Created and booted three empty Ubuntu 26.04 LTS LXD VMs on `dev-2026`:
  - `prod-control-plane` — 4 CPU, 8GiB RAM, 80GiB disk, IP `10.181.203.8`
  - `agentjack-control` — 4 CPU, 8GiB RAM, 80GiB disk, IP `10.181.203.208`
  - `agentjack-runner-pool` — 8 CPU, 16GiB RAM, 120GiB disk, IP `10.181.203.43`
- Installed base packages in all three VMs: Docker `29.1.3`, Docker Compose `2.40.3`, SSH, qemu guest agent, Python, Node/npm.
- Verified `agentjack-runner-pool` exposes `/dev/kvm`; Firecracker-in-runner-VM is viable.
- Added AgentJack state DR lane on `dev-2026`:
  - backup script: `/home/adminmatej/.local/bin/dr-agentjack-state-backup.sh`
  - restore-validation script: `/home/adminmatej/.local/bin/dr-agentjack-state-restore-validate.sh`
  - timer: `dr-agentjack-state-backup.timer` every 15 minutes
  - timer: `dr-agentjack-state-restore-validate.timer` daily 04:25
  - local encrypted artifact: `/home/adminmatej/backups/dr-files/agentjack/2026-04-29/082749/agentjack-state_dev-2026_2026-04-29_082749.tar.gpg`
  - standby copy: `mail-tba:/home/adminmatej/dr-backups/files/agentjack/2026-04-29/agentjack-state_dev-2026_2026-04-29_082749.tar.gpg`
  - restore validation passed immediately after first backup.
- Baseline LXD snapshots were captured for all three empty VMs: `baseline-ubuntu2604-empty-20260429` on `prod-control-plane`, `agentjack-control`, and `agentjack-runner-pool`.
- Staged `/opt/kraliki/` skeleton directories plus this runbook under `/opt/kraliki/runbooks/` inside all three new VMs.
- Captured Ubuntu 26.04 VM baseline artifact: `management/dev2026-ubuntu2604-vm-baseline-2026-04-29.txt`.
- Defined permanent AgentJack/Kraliki deployment homes and created filesystem skeletons in all three VMs; see `management/AGENTJACK-KRALIKI-PERMANENT-HOME-2026-04-29.md`.
- No traffic migration, DNS changes, Stripe/payment setting changes, or destructive cleanup were performed.

## Current live production map

| Surface | Current runtime | Boundary | Notes |
|---|---|---:|---|
| `identity.verduona.dev` | `dev-2026` host Docker compose (`infra/zitadel`) | host/naked | Zitadel + `zitadel-db`; Tier 0 logical DB backup already exists. |
| `hub.verduona.dev` | `central-hub` LXD container | LXC | Kraliki Hub Docker compose with `kraliki-hub`, `kraliki-hub-db`, RabbitMQ, admin UI. Reports `environment=staging`. |
| `hub.kraliki.com` | Fly.io | managed | Still points to stale/parallel Fly Hub. Canonical ownership must be decided before cutover. |
| `agentjack-dev.verduona.dev` | `dev-2026` user systemd | host/naked | `agentjack-ag-webui.service` + `agentjack-gateway.service`; now has dedicated file/state DR backup. |
| `councilnow.com` / CouncilNow app | `advisory-council` LXD + Docker | LXC | Lowest urgency; already isolated enough for first paid pilots. |
| `kraliki.com` / `www` | `mail-tba` | VM | Marketing root still on mail VM; future static hosting can move separately. |
| `mail-tba` | Hetzner VM `91.99.176.2` | VM | Active-passive DR target + current mail/CRM/marketing host. |

## New empty VM boundaries

| VM | Role | IP | Baseline state |
|---|---|---:|---|
| `prod-control-plane` | Accounts/auth/payments: Zitadel, Hub, Stripe webhooks, entitlement API | `10.181.203.8` | Ubuntu 26.04 LTS; running; Docker/Compose installed; no production services yet. |
| `agentjack-control` | AgentJack web/control plane: WebUI, gateway/API, metadata DB, scheduler | `10.181.203.208` | Ubuntu 26.04 LTS; running; Docker/Compose installed; no production services yet. |
| `agentjack-runner-pool` | Agent execution isolation: Docker/LXD now, Firecracker before untrusted self-serve | `10.181.203.43` | Ubuntu 26.04 LTS; running; Docker/Compose installed; `/dev/kvm=yes`. |

All three carry metadata:

```text
user.role=production-boundary
user.created_by=TBA-One-PA-2026-04-29
security.secureboot=false
```


## Permanent deployment homes

AgentJack is currently dogfooded/developed on Matej's Mac and still served publicly from a naked `dev-2026` host user-service path. That is not the long-term production deploy model.

Permanent production homes now exist as skeletons:

| VM | Production home | Purpose |
|---|---|---|
| `prod-control-plane` | `/srv/kraliki/{zitadel,kraliki-hub,control-plane-edge}` | Kraliki auth/API/billing/entitlements core. |
| `agentjack-control` | `/srv/kraliki/agentjack` | AgentJack product surface, gateway/API, metadata, scheduler. |
| `agentjack-runner-pool` | `/srv/kraliki/agentjack-runner` | Isolated execution plane; private only. |

Shared convention: `/srv/kraliki` for deployables, `/etc/kraliki` for root-only config/secrets pointers, `/var/lib/kraliki` for state, `/var/log/kraliki` for logs, `/opt/kraliki/runbooks` for no-secret runbooks.

## Backup and restore gates

### Already green

- Tier 0 DB lane exists for Zitadel and CouncilNow DBs:
  - `dr-postgres-backup-tier0.timer`
  - `dr-postgres-restore-validate.timer`
- AgentJack state lane now exists and passed first validation:
  - encrypted checksum OK
  - decrypted archive readable
  - required paths present:
    - `/home/adminmatej/.agentjack/`
    - `/home/adminmatej/github/agentjack/axis-workspace/`
    - user systemd units for AG WebUI + gateway

### Still required before moving services

1. Take final logical DB dumps immediately before any service quiesce.
2. Snapshot/export the target VM before first service import.
3. After service import, add VM image/export backup for:
   - `prod-control-plane`
   - `agentjack-control`
   - `agentjack-runner-pool` once runner state matters
4. Add Hub DB from `central-hub` to Tier 0/1 backup scope once canonical Hub is declared.

## Migration order with gates

### Phase 0 — current prep state ✅

- VM shells exist and are reachable.
- Base stack exists.
- AgentJack state backup exists.
- No production traffic points at the new VMs.

### Phase 1 — lock Hub canonical source [GO/NO-GO]

Decision needed: which Hub is canonical for customers/billing/users?

- Candidate A: `hub.verduona.dev` / `central-hub` on `dev-2026`.
- Candidate B: `hub.kraliki.com` / Fly.io.

Recommended default unless evidence contradicts it: use `central-hub` on `dev-2026` as canonical because it is the live control plane that currently reports Stripe billing active from `/v1/billing/status`. Treat Fly Hub as legacy/stale until proven otherwise.

Gate checks:

```bash
curl -fsS https://hub.verduona.dev/health | jq .
curl -fsS https://hub.verduona.dev/v1/billing/status | jq .
fly status -a kraliki-hub  # read-only, if Fly auth is available
```

### Phase 2 — stage `prod-control-plane` internally only

Target:

- copy redacted compose templates/config into `/opt/kraliki/prod-control-plane/`
- restore test copies of Zitadel + Hub DB into isolated internal services
- keep public Traefik unchanged
- verify internally from host and VM only

Required internal checks:

```bash
lxc exec prod-control-plane -- docker ps
lxc exec prod-control-plane -- curl -fsS http://127.0.0.1:<zitadel-port>/.well-known/openid-configuration
lxc exec prod-control-plane -- curl -fsS http://127.0.0.1:<hub-port>/health
```

GO only if internal OIDC discovery and Hub health work with restored data.

### Phase 3 — migrate `identity.verduona.dev` host route to `prod-control-plane`

This is still not `accounts.kraliki.com` cutover. It only moves existing `.dev` route from host service to VM after internal proof.

Preflight:

- fresh `zitadel-db` dump
- host route config backup
- old host Zitadel service left available for rollback

External verification:

```bash
curl -fsS https://identity.verduona.dev/.well-known/openid-configuration | jq .issuer
```

Rollback:

- revert host Traefik route to old backend
- restart old host compose/service

### Phase 4 — migrate Hub route to `prod-control-plane`

After Zitadel is stable:

- fresh `central-hub` DB dump
- restore into `prod-control-plane`
- start Hub and RabbitMQ there
- route `hub.verduona.dev` to VM
- keep Fly `hub.kraliki.com` untouched until final DNS checkpoint

External verification:

```bash
curl -fsS https://hub.verduona.dev/health | jq .
curl -fsS https://hub.verduona.dev/v1/billing/status | jq .
```

### Phase 5 — migrate AgentJack to `agentjack-control`

Preflight:

- AgentJack backup + restore validation passing within 30 minutes
- copy `/home/adminmatej/.agentjack`, `axis-workspace`, service units/config
- start services inside VM on internal ports
- route only `agentjack-dev.verduona.dev` after internal proof

External verification:

```bash
curl -fsS https://agentjack-dev.verduona.dev/readyz
curl -fsS https://agentjack-dev.verduona.dev/health || true
```

Rollback:

- revert Traefik route
- restart host user services

### Phase 6 — runner pool activation

- keep control plane isolated from execution plane
- run beta/trusted jobs in Docker/LXD runners
- require Firecracker/equivalent before arbitrary untrusted customer workloads
- no broad host secrets in runner pool; only scoped ephemeral credentials

Proof requirement:

- one runner task lifecycle: scheduled → started → isolated output → destroyed
- resource limits observed
- logs captured without secret leakage

### Phase 7 — `kraliki.com` DNS cutover [requires Matej explicit go]

Do not do this automatically.

Records to add only after previous phases are green:

```text
accounts.kraliki.com  -> 138.201.54.142 / prod-control-plane Zitadel
api.kraliki.com       -> 138.201.54.142 / prod-control-plane Hub
agentjack.kraliki.com -> 138.201.54.142 / agentjack-control
```

Explicitly do **not** put auth/API on bare `kraliki.com`.

### Phase 8 — hardening and cleanup

- add VM image exports + offsite retention
- remove/disable old host services only after burn-in
- remove `*.verduona.dev` wildcard only after explicit DNS inventory and approval
- retire Fly Hub only after canonical data decision and no active traffic/writes

## Hub canonical-source check — 2026-04-29 08:40

Fresh public checks show:

- `https://hub.verduona.dev/health` returns healthy Kraliki Hub with DB `ok`, role `shared_control_plane`, and primary consumer `CouncilNow`.
- `https://hub.verduona.dev/v1/billing/status` returns Stripe billing `active/ok` (publishable key intentionally not copied here).
- `hub.kraliki.com` currently CNAMEs to `kraliki-hub.fly.dev`, but the target does not resolve publicly during this check.

Operational conclusion: treat `central-hub` / `hub.verduona.dev` as the de facto canonical Hub for migration unless Matej explicitly says the dead Fly lane contains state we still need.

## Open blockers

| Blocker | Owner | Needed decision/action |
|---|---|---|
| Hub split brain | Matej + TBA-One-PA | Mostly resolved by live check: `central-hub` is de facto canonical; confirm only if Fly state must be recovered. |
| DNS cutover | Matej | Explicit go/no-go for `accounts.kraliki.com`, `api.kraliki.com`, `agentjack.kraliki.com`. |
| Payment settings | Matej | No Stripe/billing setting changes unless explicitly approved. |
| VM image exports | TBA-One-PA | Add after services are staged; not useful for empty VMs beyond baseline snapshots. |
| Firecracker implementation | AgentJack product lane | Required before untrusted/self-serve workloads, not before trusted beta. |

## Evidence references

- Live inventory: `/Users/matejhavlin/github/management/infra-live-inventory-2026-04-29.txt`
- dev-2026 inventory: `/Users/matejhavlin/github/management/dev2026-live-inventory-2026-04-29.txt`
- CouncilNow/AgentJack inventory: `/Users/matejhavlin/github/management/councilnow-agentjack-runtime-inventory-2026-04-29.txt`
- Topology decision: `/Users/matejhavlin/github/management/PRODUCT-INFRA-TOPOLOGY-DECISION-2026-04-29.md`
- Infra audit packet: `/Users/matejhavlin/.openclaw/workspace/claude-infra/migration-packet-2026-04-29.md`
