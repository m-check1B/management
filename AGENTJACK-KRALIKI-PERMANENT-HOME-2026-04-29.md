# AgentJack / Kraliki Permanent Deployment Home

_Date: 2026-04-29 09:25 Europe/Prague_  
_Status: target deployment home defined and skeleton created; no service cutover executed_

## Decision

AgentJack and the Kraliki ecosystem now have a permanent production home on `dev-2026` inside Ubuntu 26.04 LTS LXD VMs.

Matej's Mac stays the development/dogfood environment. It is not the production deployment home.

## Physical / VM boundaries

| Boundary | Permanent role | IP | Public target after checkpoint |
|---|---|---:|---|
| `prod-control-plane` | Kraliki core accounts/auth/payments control plane | `10.181.203.8` | `accounts.kraliki.com`, `api.kraliki.com` |
| `agentjack-control` | AgentJack product control plane | `10.181.203.208` | `agentjack.kraliki.com` |
| `agentjack-runner-pool` | AgentJack isolated execution plane | `10.181.203.43` | none directly; private control-plane scheduling only |

All three are Ubuntu 26.04 LTS and have baseline snapshot `baseline-ubuntu2604-empty-20260429`.

## Filesystem convention

This is the production convention for Kraliki/AgentJack services:

- `/srv/kraliki/<service>` — deployable code, compose projects, release artifacts
- `/etc/kraliki/<service>` — root-only runtime config and secrets pointers; never committed
- `/var/lib/kraliki/<service>` — persistent runtime state/data
- `/var/log/kraliki/<service>` — service logs
- `/opt/kraliki/runbooks` — runbooks and migration notes only, no secrets

## Service homes created

### `prod-control-plane`

- `/srv/kraliki/zitadel`
- `/srv/kraliki/kraliki-hub`
- `/srv/kraliki/control-plane-edge`
- `/etc/kraliki/{zitadel,kraliki-hub,control-plane-edge}`
- `/var/lib/kraliki/{zitadel,kraliki-hub,control-plane-edge}`
- `/var/log/kraliki/{zitadel,kraliki-hub,control-plane-edge}`

### `agentjack-control`

- `/srv/kraliki/agentjack/source`
- `/srv/kraliki/agentjack/releases`
- `/srv/kraliki/agentjack/shared`
- `/srv/kraliki/agentjack/compose`
- `/etc/kraliki/agentjack`
- `/var/lib/kraliki/agentjack/home` — replacement for current `.agentjack` state
- `/var/lib/kraliki/agentjack/workspaces` — replacement for current workspace/runtime state
- `/var/lib/kraliki/agentjack/db`
- `/var/lib/kraliki/agentjack/uploads`
- `/var/log/kraliki/agentjack`

### `agentjack-runner-pool`

- `/srv/kraliki/agentjack-runner/source`
- `/srv/kraliki/agentjack-runner/images`
- `/srv/kraliki/agentjack-runner/compose`
- `/etc/kraliki/agentjack-runner`
- `/var/lib/kraliki/agentjack-runner/workspaces`
- `/var/lib/kraliki/agentjack-runner/tasks`
- `/var/lib/kraliki/agentjack-runner/cache`
- `/var/lib/kraliki/agentjack-runner/microvms`
- `/var/log/kraliki/agentjack-runner`

Each VM also has `/opt/kraliki/runbooks/PERMANENT-HOME.md` documenting its local role.

## Deployment rule

New production AgentJack/Kraliki deployment work should target these homes, not Matej's Mac and not naked `dev-2026` host services.

Current live services stay where they are until explicit migration checkpoints:

- `identity.verduona.dev` remains live on current host-level Zitadel until `prod-control-plane` is internally proven.
- `hub.verduona.dev` remains live on `central-hub` until Hub canonical-source and restore checks are done.
- `agentjack-dev.verduona.dev` remains live on the naked host until AgentJack is imported and verified in `agentjack-control`.

## Next deploy work for AgentJack

1. Convert the current Mac/dev AgentJack runtime assumptions into a production compose/systemd lane under `/srv/kraliki/agentjack/compose`.
2. Map current state:
   - `.agentjack` -> `/var/lib/kraliki/agentjack/home`
   - `axis-workspace` / runtime workspaces -> `/var/lib/kraliki/agentjack/workspaces`
   - logs -> `/var/log/kraliki/agentjack`
3. Add environment/config placeholders under `/etc/kraliki/agentjack` without copying secrets into docs.
4. Start internal-only AgentJack on `agentjack-control` and verify `/health`, `/readyz`, login/OIDC, workspaces, and a minimal task lifecycle.
5. Only after proof, move the public route from `agentjack-dev.verduona.dev`; `agentjack.kraliki.com` remains a later explicit DNS checkpoint.

## Guardrails

- No DNS cutover without explicit go/no-go.
- No payment or billing setting changes without Matej approval.
- No destructive cleanup of Mac/host AgentJack state until restore and rollback paths are proven.
- Untrusted/self-serve arbitrary workloads require Firecracker or equivalent microVM isolation in `agentjack-runner-pool`.
