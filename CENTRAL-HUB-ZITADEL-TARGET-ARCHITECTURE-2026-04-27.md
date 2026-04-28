# Central Hub + Zitadel Target Architecture

_Date: 2026-04-27_
_Owner: Axis_
_Status: target decision + execution plan_

## Executive decision

The central setup should be:

- **Zitadel (`identity.verduona.dev`) = identity provider only.** It owns login methods, MFA/passkeys/social login, email verification, and OIDC identity assertions.
- **Hub (`hub.verduona.dev`) = Kraliki/Verduona control plane.** It owns product users, tenants, memberships, entitlements, billing, usage, product access, audit logs, service tokens, and auth brokering for all products.
- **Products (CouncilNow, AgentJack, future apps) = thin Hub clients.** They keep only product-specific data and reference `hub_user_id` / `hub_tenant_id`. They do not fork auth, billing, tenant membership, or Stripe logic.

This avoids three failure modes already visible in the system: per-app Hub forks, multiple Zitadel lanes, and products reimplementing login/billing.

## Current state found

1. `identity.verduona.dev` is live and serves Zitadel OIDC discovery.
2. `hub.verduona.dev` currently returns `502 Bad Gateway`; Traefik routes it to `host.docker.internal:8203`, but no Hub process is listening on `127.0.0.1:8203`.
3. The canonical Hub database candidate in `kraliki-postgres` has `oidc_providers` table present in `kraliki_hub`, but provider count is currently `0`.
4. Hub OIDC code currently assumes provider endpoints by appending `/authorize`, `/token`, `/userinfo` to `issuer_url`. Zitadel discovery exposes `/oauth/v2/authorize`, `/oauth/v2/token`, `/oidc/v1/userinfo`, so the implementation should use discovery metadata instead of path guessing.
5. Hub OIDC discovery currently hardcodes `https://hub.kraliki.com`; target public Hub for the current infra lane is `https://hub.verduona.dev`.
6. CouncilNow is production-green today because it has a working app-local/hub-fork auth + billing path. That should stay stable while central Hub is rebuilt, then migrate behind an adapter.
7. There are multiple scattered Hub/admin codebases under `/home/adminmatej/github`; do not implement blindly in the first one found.

## Scattered codebase inventory

### Canonical candidates

| Path | Role | Current classification |
|---|---|---|
| `/home/adminmatej/github/applications/kraliki-hub` | Standalone FastAPI Hub repo | **Best central Hub backend candidate.** Separate git repo, latest observed commit `539d189` (`chore: sync server changes`, 2026-04-27). Contains newer standalone-only pieces such as `app/clients`, `app/routers/council.py`, `app/routers/sandbox.py`, and standalone DB/test artifacts. Differs materially from monorepo `apps/hub`; reconcile before declaring it production source of truth. |
| `/home/adminmatej/github/kraliki-monorepo/apps/hub` | Monorepo FastAPI Hub copy | Legacy/product-monorepo Hub backend. Active history exists, but it differs from standalone `applications/kraliki-hub`. Useful as source material and for old automation context, not safe to treat as canonical without diff review. |
| `/home/adminmatej/github/kraliki-monorepo/apps/hub-admin` | SvelteKit Hub Admin app | **Canonical admin UI source candidate.** SvelteKit admin dashboard with `/login`, `/dashboard`, `/tenants`, `/users`, `/customers`, `/billing`, `/licensing`, `/instances`, `/byok`, `/monitoring`. Designed to run on `:8280` and proxy Hub backend at `HUB_URL` default `http://127.0.0.1:8203`. |

### Duplicate / fork copies

| Path | Notes |
|---|---|
| `/home/adminmatej/github/applications/kraliki-monorepo/apps/hub` | Mostly same as `/home/adminmatej/github/kraliki-monorepo/apps/hub`, with local runtime files (`.env`, secrets, SQLite/test DB artifacts). |
| `/home/adminmatej/github/applications/kraliki-monorepo-automation/apps/hub` | Exact observed copy of `/home/adminmatej/github/kraliki-monorepo/apps/hub` after excluding build/runtime artifacts. Automation mirror, not a new authority. |
| `/home/adminmatej/github/applications/kraliki-monorepo/apps/hub-admin` | Same Hub Admin app plus local `package-lock.json`; not a distinct product. |
| `/home/adminmatej/github/applications/kraliki-monorepo-automation/apps/hub-admin` | Exact observed copy of top-level `apps/hub-admin`. Automation mirror. |
| `/home/adminmatej/github/applications/advisory-council-app/backend/hub-fork` | CouncilNow app-local Hub fork. Keep stable while central Hub is rebuilt, then migrate behind product adapter. Do not break it during central Hub work. |
| `/home/adminmatej/github/applications/production-hub` | SvelteKit dashboard for repo-local `open-kraliki` autodevelopment packages, port `3001`. Despite the name, this is **not** the central identity/billing Hub and should not be used for Zitadel/Hub auth work. |

### Hub Admin details

The admin app already exists. Current design:

- Runtime: SvelteKit 5, adapter-node, `node build/index.js`.
- Intended process name: `hub-admin`.
- Intended port: `8280`.
- Intended backend dependency: central Hub on `HUB_URL`, default `http://127.0.0.1:8203`.
- Session: signed HttpOnly cookie `hub_admin_session` using `HUB_ADMIN_SESSION_SECRET`.
- Login: `POST {HUB_URL}/api/v1/auth/login`, then requires `is_admin=true` in response or JWT claims.
- API pattern: SvelteKit server proxies admin routes to Hub with `Authorization: Bearer {admin_jwt}`.
- Existing routing docs: `infra/PUBLIC_ENDPOINTS.md` lists `hub.verduona.dev -> :8203` and `hub-admin.verduona.dev -> :8280`; current dev-2026 inspection found no local listeners on `:8203` or `:8280`.

Decision: central Hub restoration should wire both surfaces deliberately: **Hub API on `8203`; Hub Admin on `8280`; admin UI must authenticate through Hub, not directly through Zitadel.**

## Target domains

| Domain | Role | Notes |
|---|---|---|
| `identity.verduona.dev` | Zitadel IdP | One canonical Zitadel instance backed by canonical `zitadel-db`. |
| `hub.verduona.dev` | Central Hub API/auth/billing/control plane | Public API, auth broker, JWKS, entitlements, admin endpoints. |
| `auth.verduona.dev` | Optional alias later | Can point to Hub auth UI if we want cleaner login branding. Not required for v1. |
| `councilnow.com` | Product | Uses Hub as auth/billing source. |
| `agentjack-dev.verduona.dev` / `agentjack.kraliki.com` | Product | Uses Hub as auth/billing/source-of-truth. |

## Auth model

### Correct login flow

Do **not** pass long-lived JWTs in query strings and do **not** depend on `localStorage` for production auth.

1. Product sends user to:
   - `https://hub.verduona.dev/auth/login?product=agentjack&return_to=...`
2. Hub creates a state + nonce + PKCE verifier stored server-side.
3. Hub redirects to Zitadel using discovery metadata:
   - `authorization_endpoint` from `https://identity.verduona.dev/.well-known/openid-configuration`
4. Zitadel authenticates user through password, Google, passkey, or MFA.
5. Zitadel redirects back to Hub callback.
6. Hub validates ID token/JWKS/nonce, optionally calls `userinfo`, then upserts:
   - Hub user
   - tenant memberships
   - product access grants
7. Hub issues a **short-lived one-time product auth code** and redirects back to the product.
8. Product backend exchanges that code with Hub server-to-server.
9. Product sets its own `HttpOnly; Secure; SameSite=Lax` session cookie.
10. Product calls Hub for `/me`, entitlements, tenant context, billing, and usage.

### Why this shape

- Works across unrelated domains (`councilnow.com`, `verduona.dev`, `kraliki.com`).
- Avoids shared-cookie assumptions.
- Avoids leaking JWTs in URLs.
- Keeps Zitadel vendor-specific details out of products.
- Lets products stay independently deployable while sharing identity and billing.

## Hub responsibilities

Hub owns:

- Users and external identity links (`zitadel_sub`, email, verified state)
- Tenants, organizations, workspaces, memberships, roles
- Product grants (`agentjack`, `councilnow`, future verticals)
- Entitlements and plan aliases
- Stripe customers, subscriptions, checkout, portal, webhooks, usage metering
- Service tokens, pairing codes, channel-link auth
- Audit log for login, billing, admin, token, and membership changes
- JWKS/public keys for product token verification
- Admin/control-plane UI or API for operators

Hub does **not** own product-specific runtime state like AgentJack conversations or CouncilNow council runs.

## Zitadel responsibilities

Zitadel owns:

- Password/passkey/social authentication
- MFA policy
- Email verification policy
- OIDC clients
- Identity lifecycle at IdP layer

Zitadel does **not** own product plans, tenants, Stripe, app roles, or vertical entitlements. Those stay in Hub.

## Data ownership

| Data | Canonical owner |
|---|---|
| IdP identity, credentials, MFA | Zitadel DB (`zitadel-db`) |
| Product user record | Hub DB (`hub-db` / `kraliki_hub`) |
| Tenant/member/role | Hub DB |
| Billing/customer/subscription | Hub DB + Stripe |
| Product session/council/chat data | Product DB keyed by Hub IDs |
| Runtime/integration health | Product runtime + Hub summaries |

## Deployment target

For the current dev-2026 lane:

- Run Hub API as a supervised service bound to `127.0.0.1:8203` or a container published to `127.0.0.1:8203`.
- Run Hub Admin as a supervised SvelteKit service bound to `127.0.0.1:8280` or a container published to `127.0.0.1:8280`.
- Keep public routes explicit:
  - `hub.verduona.dev -> Hub API :8203`
  - `hub-admin.verduona.dev -> Hub Admin :8280`
- Back Hub by a canonical Postgres database, not SQLite and not app-local forks.
- Include Hub DB + Zitadel DB in Tier 0 DR backup policy to `mail-tba`.

Recommendation: use **systemd user service or Docker Compose** for Hub API. PM2 is acceptable for Hub Admin/app dev servers, but the central API/control plane should behave like infra.

## Migration plan

### Phase 0 — Freeze working product paths

- Keep CouncilNow current auth/billing path green.
- Keep AgentJack dev path available.
- Do not cut over products until Hub has its own proof.

### Phase 1 — Restore central Hub

Acceptance:

- `curl https://hub.verduona.dev/health` returns `200`.
- `curl https://hub-admin.verduona.dev/login` returns `200`.
- Health body reports shared-control-plane role.
- Hub API process is supervised and restarts cleanly.
- Hub Admin process is supervised and restarts cleanly.
- Hub Admin can log in using a Hub admin user and load `/dashboard` without 502.
- Hub uses production Postgres and EdDSA/JWKS keys.

### Phase 2 — Fix Hub ↔ Zitadel OIDC properly

Required code changes:

- Add configurable `HUB_PUBLIC_URL` / issuer; stop hardcoding `hub.kraliki.com`.
- Add OIDC discovery fetch/cache per provider.
- Store state/nonce/PKCE in Redis or DB, not in-memory dict for production.
- Use Zitadel discovery endpoints, not guessed `/authorize` paths.
- Validate ID token issuer/audience/nonce/signature via JWKS.
- Register provider seed for `zitadel`.

Acceptance:

- `GET /auth/api/v1/auth/oidc/authorize/zitadel` redirects to `identity.verduona.dev/oauth/v2/authorize`.
- Password-only Zitadel login works.
- Google-through-Zitadel login works.
- Hub `/me` returns the same user identity after login.

### Phase 3 — Product auth adapter

- Products redirect to Hub login.
- Products exchange one-time auth codes server-side.
- Products set HttpOnly sessions.
- Remove production dependence on localStorage auth tokens.

Acceptance:

- CouncilNow and AgentJack both log into the same Hub user.
- Product DB rows are keyed by `hub_user_id` / `hub_tenant_id`.
- Logout clears product session and optionally calls Hub/Zitadel end-session.

### Phase 4 — Centralize billing/entitlements

- Hub remains the only Stripe integration owner.
- Products ask Hub for entitlement state and report usage events.
- Existing CouncilNow checkout proof remains green after adapter cutover.

Acceptance:

- Starter/pro/scale/enterprise checkout paths still return live Stripe checkout URLs.
- Portal and subscription endpoints work through Hub.
- Product access gates read Hub entitlements, not local tier copies.

### Phase 5 — Retire drift

- Retire `advisory-zitadel` after confirming no production traffic.
- Retire app-local hub forks only after each product has passed Hub adapter proof.
- Delete/disable obsolete `hub.kraliki.com` assumptions or make it a deliberate alias.

## Non-negotiable acceptance tests

1. `https://hub.verduona.dev/health` -> `200`
2. `https://identity.verduona.dev/.well-known/openid-configuration` -> issuer `https://identity.verduona.dev`
3. Hub OIDC `authorize/zitadel` redirects to the discovery-provided Zitadel authorization endpoint.
4. One account can authenticate and access both CouncilNow and AgentJack.
5. Tenant switch changes Hub-issued tenant context and does not leak product data across tenants.
6. Stripe checkout + portal + webhook updates entitlements in Hub.
7. Product sessions use HttpOnly cookies, not production localStorage tokens.
8. Hub DB and Zitadel DB are covered by encrypted DR backup + restore validation.

## Immediate next implementation packet

**Title:** Restore Central Hub + Zitadel broker proof

**Scope:**
- Choose and reconcile the canonical Hub backend codebase, starting from `/home/adminmatej/github/applications/kraliki-hub` and diffing against `/home/adminmatej/github/kraliki-monorepo/apps/hub` before implementation.
- Start/supervise central Hub API on dev-2026 at `127.0.0.1:8203`.
- Start/supervise Hub Admin from `/home/adminmatej/github/kraliki-monorepo/apps/hub-admin` at `127.0.0.1:8280`, pointed at `HUB_URL=http://127.0.0.1:8203`.
- Patch Hub OIDC provider implementation to use discovery metadata.
- Add provider seed/config for Zitadel.
- Add tests for discovery endpoint resolution, callback validation, state expiry persistence, and Hub Admin login/dashboard proxy behavior.
- Prove live `hub.verduona.dev` health, `hub-admin.verduona.dev/login`, and `authorize/zitadel` redirect.

**Out of scope:**
- Cutting over CouncilNow or AgentJack product auth.
- Changing Stripe live settings beyond reading existing config.
- Any billing/payment settings changes.

## Decision summary

The perfect setup is **one Zitadel, one central Hub, many thin products**.

Zitadel authenticates. Hub authorizes, bills, tenants, and brokers. Products consume Hub and keep their own product data only.
