# CouncilNow Evidence Brain Deploy — 2026-04-26

## Deploy

Deployed CouncilNow commit `3121dd9 Add CouncilNow Evidence Brain MVP slice` to the public dev-2026 `advisory-council` LXC container.

Deploy tag:

```text
evidence-brain-mvp-202604261525
```

Command:

```bash
DEPLOY_TAG=evidence-brain-mvp-202604261525 ./deploy.sh dev-2026
```

## Build gates before deploy

```text
backend tests: 70 passed
frontend npm run check: passed
frontend npm run build: passed
docker compose config --quiet: passed
git diff --check: passed
```

## Container state after deploy

Inside `dev-2026` → LXC `advisory-council`:

```text
advisory-council-council-api-1      advisory-council-api:evidence-brain-mvp-202604261525       healthy  0.0.0.0:8210->8210
advisory-council-council-hub-1      advisory-council-hub:evidence-brain-mvp-202604261525       healthy  0.0.0.0:8211->8211
advisory-council-council-frontend-1 advisory-council-frontend:evidence-brain-mvp-202604261525  up       0.0.0.0:5180->5180
open-notebook sidecar               lfnovo/open_notebook:v1-latest                             healthy  127.0.0.1:5055/8502 only
open-notebook-surrealdb             surrealdb/surrealdb:v2                                     up       127.0.0.1:8001 only
```

Local container health:

```json
{"status":"ok","version":"0.4.0","database":"ok"}
```

## Public smoke test

Public URL:

```text
https://councilnow.com
```

Fresh production account:

```text
evidence-deploy-1777210071097@gmail.com
```

Checks:

```text
GET  /health                         -> 200
GET  /app                            -> 200, HTML present
POST /auth/api/v1/auth/register      -> 201
POST /auth/api/v1/auth/login         -> 200
GET  /auth/api/v1/auth/me            -> 200, Stripe customer cus_UPHHjEG7Bocas2
```

Evidence Brain public API smoke:

```text
POST /api/v1/evidence/sources agenda-minutes.md   -> 200, 3 claims, 0 contradictions
POST /api/v1/evidence/sources budget.csv          -> 200, 1 claim, 0 contradictions
POST /api/v1/evidence/sources contradiction.md    -> 200, 2 claims, 2 contradictions
GET  /api/v1/evidence/claims                      -> 200, 6 claims
GET  /api/v1/evidence/contradictions              -> 200, 2 contradictions
POST /api/v1/evidence/packets                     -> 200, 6 claims, 2 contradictions, sha256 citations present
```

Expected contradictions detected publicly:

```text
Public contribution conflict: EUR 500,000 vs EUR 650,000
Traffic mitigation conflict: required vs optional/deferred
```

Evidence Notebook public proxy smoke:

```text
GET /api/v1/research-notebook/status -> 200
configured=true, reachable=true, url=http://open-notebook:5055, auth_configured=true
```

## Result

The new CouncilNow version is publicly visible and the Evidence Brain MVP flow is live on `https://councilnow.com`.
