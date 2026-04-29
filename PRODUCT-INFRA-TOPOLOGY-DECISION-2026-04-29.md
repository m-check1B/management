# Product Infra Topology Decision — CouncilNow + AgentJack

_Date: 2026-04-29 08:10 Europe/Prague_
_Last updated: 2026-04-29 08:36 Europe/Prague_
_Status: accepted execution direction / reversible boundaries created; no DNS/payment cutover_


## Execution update — 2026-04-29 08:36

Reversible isolation boundaries now exist on `dev-2026`:

| VM | Spec | IP | State |
|---|---|---:|---|
| `prod-control-plane` | 4 CPU / 8GiB / 80GiB | `10.181.203.8` | Ubuntu 26.04 LTS; running, Docker/Compose installed |
| `agentjack-control` | 4 CPU / 8GiB / 80GiB | `10.181.203.208` | Ubuntu 26.04 LTS; running, Docker/Compose installed |
| `agentjack-runner-pool` | 8 CPU / 16GiB / 120GiB | `10.181.203.43` | Ubuntu 26.04 LTS; running, Docker/Compose installed, `/dev/kvm=yes` |

AgentJack state DR backup was added before migration work:

- local encrypted path: `/home/adminmatej/backups/dr-files/agentjack/.../*.tar.gpg`
- remote standby path: `mail-tba:/home/adminmatej/dr-backups/files/agentjack/.../*.tar.gpg`
- timers: `dr-agentjack-state-backup.timer` every 15 minutes and `dr-agentjack-state-restore-validate.timer` daily
- first backup and restore validation passed at 2026-04-29 08:27 CEST

Still not done: public route migration, DNS cutover, Stripe/payment changes, destructive cleanup.

## Decision summary

1. **Core account/payment control plane** gets its own boring production VM on dev-2026.
2. **CouncilNow** remains a SaaS-style product deployment: one product runtime boundary, shared app DB, tenants/customers as rows and subscriptions, scale horizontally by API/frontend/worker containers.
3. **AgentJack** must not stay naked on the dev-2026 host. It gets its own product control-plane VM plus a separate runner pool.
4. **AgentJack runner pool** starts as hardened containers for trusted/beta dogfood, but the architecture must be Firecracker-ready. Firecracker/microVM runners become mandatory before untrusted self-serve customer workloads or arbitrary tool/code execution.
5. **mail-tba/small Hetzner VM** remains hot/warm DR standby and backup/restore target, not normal writer.

## Live state checked

- CouncilNow currently runs in LXD container `advisory-council` on dev-2026, with Docker services:
  - `council-frontend`
  - `council-hub`
  - `council-api`
  - `council-router`
  - `council-db`
  - Open Notebook sidecars
- AgentJack currently runs directly on dev-2026 user systemd services:
  - `agentjack-ag-webui.service`
  - `agentjack-gateway.service`
- Therefore CouncilNow already has a product boundary; AgentJack currently does not.

## Target chart

```text
Internet / Cloudflare
│
├── kraliki.com                  → marketing / platform trust front door
├── app.kraliki.com              → main Kraliki/AgentJack app shell later
├── accounts.kraliki.com         → Core Control Plane / Zitadel
├── api.kraliki.com              → Core Control Plane / Hub API
├── councilnow.com               → CouncilNow marketing/app split
└── agentjack.kraliki.com        → AgentJack product UI / marketing/app split

Hetzner dev-2026 physical host
│
├── prod-control-plane VM        [BORING / CUSTOMER-CRITICAL]
│   ├── Zitadel                  → auth / OIDC
│   ├── Zitadel Postgres         → identity DB
│   ├── Kraliki Hub              → users, tenants, billing, entitlements
│   ├── Hub Postgres             → user/billing/product-access DB
│   ├── Stripe webhooks          → subscription state ingest
│   └── minimal monitoring/exporters
│
├── councilnow VM/LXC            [SAAS PRODUCT RUNTIME]
│   ├── council frontend
│   ├── council API
│   ├── council app/hub adapter
│   ├── council DB
│   ├── model router
│   └── workers/queue later
│
├── agentjack-control VM/LXC     [PRODUCT CONTROL PLANE]
│   ├── AG WebUI
│   ├── AgentJack API/gateway
│   ├── workspace/session metadata DB
│   ├── scheduler / runner allocator
│   └── runner queue
│
├── agentjack-runner-pool        [CUSTOMER WORKLOAD ISOLATION]
│   ├── beta/trusted: Docker/LXD containers
│   ├── production self-serve: Firecracker microVMs
│   ├── per tenant/workspace/session resource limits
│   └── no broad host secrets; only scoped ephemeral tokens
│
└── garage / experiments / OpenClaw / Axis / dogfood
    └── explicitly outside boring production core

mail-tba / small Hetzner VM
└── DR standby
    ├── encrypted DB backup copies
    ├── restore validation
    ├── optional warm Postgres standby after boring-green backups
    └── manual/controlled failover target
```

## CouncilNow scaling model

CouncilNow is a normal SaaS app:

- users/tenants/sessions/subscriptions are rows in DB
- no customer-specific runtime by default
- scale by adding API workers, frontend replicas, queue workers, model-router capacity
- isolation mostly at auth/tenant/data layer
- LXC/Docker is acceptable for first customers because user input is data, not arbitrary execution

Graduation path:

1. Current LXC + Docker is acceptable for first paid pilots.
2. Move/rename into explicit `councilnow-prod` VM/LXC if clarity is needed.
3. Add worker queue and horizontal API replicas only when load requires it.
4. Firecracker not needed for core CouncilNow unless we introduce customer-uploaded code/tools or strong per-customer compute isolation.

## AgentJack scaling model

AgentJack is not just SaaS rows. It runs agents, tools, browsers, file ops, integrations, and eventually customer-controlled workflows.

Split it into two planes:

### AgentJack control plane

- web UI
- auth/session/workspace metadata
- policy/permissions
- billing/entitlement checks through Hub
- runner scheduling

This can run like a normal product service in an `agentjack-control` VM/LXC.

### AgentJack execution plane

- one runner per workspace/session/tenant/task class
- starts as Docker/LXD for trusted beta
- moves to Firecracker for untrusted/self-serve/customer workloads
- scoped credentials only
- egress and filesystem limits
- easy destroy/recreate

## Firecracker decision

Do **not** block today's infrastructure decision on Firecracker implementation.

But do lock the boundary now:

- AgentJack must talk to runners through a runner-provider abstraction.
- Provider v1: local Docker/LXD runner.
- Provider v2: Firecracker microVM runner.
- Production self-serve launch requires provider v2 or equivalent isolation.

Firecracker is most valuable for AgentJack, not CouncilNow.


## Hub canonical-source update — 2026-04-29 08:40

`hub.verduona.dev` is the de facto live Hub path: `/health` is DB-ok and `/v1/billing/status` reports Stripe billing active. `hub.kraliki.com` still CNAMEs to `kraliki-hub.fly.dev`, but that Fly hostname did not resolve publicly during the check. Unless Matej knows of unrecovered Fly-only state, migrate from `central-hub` / `hub.verduona.dev` and treat Fly Hub as stale.

## Immediate migration decisions

1. Create `prod-control-plane` VM for Kraliki accounts/payments.
2. Leave CouncilNow in its existing dedicated LXC for now; optionally rename/rebuild later as `councilnow-prod`.
3. Move AgentJack off naked dev-2026 systemd into `agentjack-control` VM/LXC.
4. Add `agentjack-runner-pool` as separate from control plane.
5. Start runner pool as containers for internal beta, but design API so Firecracker swap is mechanical.
6. Keep `mail-tba` as DR target and add VM image exports once production VMs exist.

## Recommended urgency ranking

1. Core control plane VM + domain decision (`accounts.kraliki.com`, `api.kraliki.com`).
2. AgentJack control-plane isolation — move off naked host.
3. AgentJack runner abstraction.
4. Firecracker runner implementation.
5. CouncilNow horizontal scale only when load demands it.
