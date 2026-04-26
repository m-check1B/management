# CouncilNow × Open Notebook Evaluation — 2026-04-26

## Question

Should CouncilNow integrate [Open Notebook](https://github.com/lfnovo/open-notebook) directly instead of routing users to Google / NotebookLM?

## Short answer

**Yes — we should stop treating NotebookLM as the preferred user path and integrate Open Notebook as a first-party CouncilNow research notebook.**

But we should **not** merge Open Notebook's whole Next.js app into CouncilNow in one big step. The right path is:

1. Run Open Notebook as an internal sidecar service.
2. Keep it behind CouncilNow auth and backend APIs.
3. Replace the current NotebookLM bridge UI with a `Research Notebook` / `Evidence Notebook` surface inside CouncilNow.
4. Only later consider white-labeling or embedding parts of Open Notebook's UI.

This gives us the product win — no Google routing — without turning CouncilNow into a messy cross-framework fork.

## Repository inspected

- Local clone: `/Users/matejhavlin/github/open-notebook`
- Upstream: `lfnovo/open-notebook`
- Branch/commit inspected: `main` at `ec41ef8 feat(api): add configurable CORS origins via CORS_ORIGINS (#767)`
- GitHub repo state from `gh repo view`:
  - License: MIT
  - Stars: 22,723
  - Forks: 2,629
  - Not archived
  - Last pushed: 2026-04-21T23:04:13Z

## What Open Notebook is

Open Notebook is explicitly positioned as:

> An open source, privacy-focused alternative to Google's NotebookLM.

Architecture:

- **Backend:** FastAPI on port `5055`
- **Frontend:** Next.js/React on port `8502`
- **Database:** SurrealDB on port `8000`
- **Container deploy:** `docker-compose.yml` with `surrealdb` + `open_notebook`
- **License:** MIT
- **Core model:** notebooks → sources → notes → chat/search/ask/transformations
- **AI providers:** OpenAI, Anthropic, Groq, xAI, Ollama, Mistral, Voyage, OpenRouter, Google, etc.
- **Embeddings:** requires a configured embedding provider for vector search; non-Google choices exist.

Useful APIs discovered:

- `POST /api/notebooks`
- `GET /api/notebooks`
- `POST /api/sources` / `POST /api/sources/json`
- `POST /api/search`
- `POST /api/search/ask`
- `POST /api/chat/sessions`
- `POST /api/chat/execute`
- `POST /api/chat/context`
- notes and transformations endpoints

## Current CouncilNow NotebookLM state

CouncilNow already has a NotebookLM bridge:

- Backend router: `backend/app/routers/notebooklm.py`
- Bridge service: `backend/app/services/notebooklm_bridge.py`
- Frontend client: `frontend/src/lib/notebooklm-context.ts`
- UI section: `frontend/src/routes/app/data/+page.svelte`
- Dependency: `backend/requirements.txt` includes `notebooklm-py>=0.3.4`

Current behavior:

- User pastes NotebookLM auth JSON.
- CouncilNow syncs pinned files/data/sessions out to NotebookLM using the unofficial `notebooklm` CLI.
- CouncilNow can pull source text back from NotebookLM.

This is clever, but strategically wrong for the product now: it sends users into Google's product surface and requires Google auth/session material. That conflicts with the brand promise we want.

## Fit with CouncilNow

### Strong fit

Open Notebook maps cleanly onto CouncilNow's data flow:

| CouncilNow | Open Notebook |
|---|---|
| User data files | Sources |
| Pinned files | Selected sources/context |
| Council session | Notebook or note |
| Council report | AI/manual note |
| Data tab search | Text/vector search |
| Ask against evidence | `search/ask` or chat |
| Decision Sprint evidence room | Notebook |

The conceptual fit is very good. CouncilNow needs an evidence room around every major decision. Open Notebook already has the container model for that.

### Product upside

Integrating this gives CouncilNow a stronger product story:

- Users never have to leave CouncilNow for Google NotebookLM.
- Data stays in our infrastructure / selected model providers.
- CouncilNow becomes more than a council-runner: it becomes a decision workspace with an evidence layer.
- The current `Data` tab can evolve into `Evidence Notebook` instead of a thin file manager.
- Podcast/audio features could become a future premium differentiator, but not part of MVP.

## The important caveat

Open Notebook is not currently enterprise multi-tenant in the way CouncilNow needs.

Observed security/product constraints:

- Auth is a single bearer password via `OPEN_NOTEBOOK_PASSWORD`.
- No per-user auth/RBAC inside Open Notebook itself.
- Docs explicitly call out limitations: no user management, no session timeout, no rate limiting, no audit logging.
- CORS defaults to `*` unless `CORS_ORIGINS` is set.
- Open Notebook stores API provider credentials, encrypted with `OPEN_NOTEBOOK_ENCRYPTION_KEY`, but key management/rotation is basic.

Therefore: **do not expose Open Notebook directly to customers.**

CouncilNow must be the auth boundary. Open Notebook should be internal-only, with CouncilNow mapping users/sessions to notebooks and enforcing permissions.

## Recommended architecture

### Phase 1 — Sidecar adapter, no direct user exposure

Add Open Notebook to the CouncilNow LXD Docker Compose stack as internal services:

- `open-notebook-api/ui` container, internal network only
- `open-notebook-surrealdb` volume-backed container
- Bind ports only inside Docker network or localhost, not public Traefik routes
- Set:
  - `OPEN_NOTEBOOK_PASSWORD`
  - `OPEN_NOTEBOOK_ENCRYPTION_KEY`
  - `CORS_ORIGINS=https://councilnow.com`
  - non-Google model/embedding provider defaults

Then add a CouncilNow backend service:

- `OpenNotebookBridge` or `ResearchNotebookService`
- Stores per-user mapping in CouncilNow Postgres/user storage:
  - `user_id`
  - `council_session_id` or project id
  - `open_notebook_notebook_id`
- Proxies only approved operations:
  - create/find notebook
  - push pinned files as sources
  - push council report as note
  - ask/search notebook
  - optionally import generated notes back into CouncilNow user data

### Phase 2 — Replace NotebookLM UI copy and calls

In the CouncilNow Data page:

- Rename `NotebookLM Context Bridge` → `Research Notebook` / `Evidence Notebook`
- Remove auth JSON fields and NotebookLM-specific language
- Add actions like:
  - `Create Evidence Notebook`
  - `Sync pinned files`
  - `Ask evidence base`
  - `Use in next council run`
- Keep CouncilNow visual language. Do not embed the full Open Notebook frontend yet.

### Phase 3 — Deeper product integration

Once the adapter is stable:

- Add notebook-aware council runs: every session can have an evidence notebook.
- Save council outputs back as notes.
- Use Open Notebook search/ask to build evidence packets before advisors deliberate.
- Consider white-labeling selected Open Notebook UI components or building a native Svelte version.

## What not to do

- Do **not** iframe public NotebookLM.
- Do **not** ask users to paste Google NotebookLM auth JSON.
- Do **not** expose Open Notebook's raw API/UI publicly as `notebook.councilnow.com` until multi-tenant/auth hardening is solved.
- Do **not** port the entire Next.js frontend into SvelteKit right now.
- Do **not** enable Google as the default provider inside Open Notebook if the goal is avoiding Google dependency.

## Google dependency distinction

This evaluation is about the NotebookLM/product-routing dependency.

CouncilNow still has separate Google dependencies:

- Current production auth uses direct Google OAuth.
- There is a separate optional/shadow Google Agent Platform verifier path.

Those are different decisions. Replacing NotebookLM with Open Notebook solves the user-facing research-workspace problem. It does **not** automatically remove Google OAuth from login.

If the principle is now "do not route users to Google at all," then the next follow-up is an auth plan: add email magic link and/or Zitadel-first login so Google OAuth becomes optional, not required.

## Decision

**Proceed with Open Notebook integration, but as an internal sidecar + CouncilNow-native proxy, not as a direct exposed app.**

This is the best middle path:

- Fast enough to ship.
- Aligned with the no-Google-routing product direction.
- Keeps CouncilNow UX first-party.
- Avoids adopting Open Notebook's current single-password auth model as a customer-facing security boundary.
- Leaves room to deepen integration later.

## MVP build scope

1. Add Open Notebook + SurrealDB services to CouncilNow compose, internal-only.
2. Add backend config/env for `OPEN_NOTEBOOK_URL`, `OPEN_NOTEBOOK_PASSWORD`.
3. Add `ResearchNotebookService` wrapper.
4. Implement endpoints:
   - `GET /api/v1/research-notebook/status`
   - `POST /api/v1/research-notebook/sync-out`
   - `POST /api/v1/research-notebook/ask`
   - optional `POST /api/v1/research-notebook/import-note`
5. Replace NotebookLM panel text/actions in Data page.
6. Remove or hide NotebookLM auth JSON UI.
7. Gate with tests:
   - Open Notebook client unit tests with mocked HTTP
   - route auth tests
   - frontend copy regression: no `NotebookLM`/`Google` call-to-action in user flow
   - compose config sanity check

## Recommendation label

**Green light: yes, integrate Open Notebook.**

Use it to make CouncilNow's evidence layer first-party. Keep Google out of the user path.

## Implementation pass — 2026-04-26 13:40 CEST

Local CouncilNow MVP implementation completed and pushed to `origin/master`:

- Commit: `8d134ab Add CouncilNow Evidence Notebook sidecar`
- No production deploy/restart was performed.
- Push target was `master`; current GitHub deploy workflow triggers on `main` only, so this push should not auto-deploy.

Implemented:

- Internal Open Notebook sidecar services in `docker-compose.yml`:
  - `open-notebook-surrealdb`
  - `open-notebook`
  - localhost-bound ports for UI/API/SurrealDB
  - no public Traefik route
- CouncilNow backend proxy/tenant boundary:
  - `backend/app/services/research_notebook.py`
  - `backend/app/routers/research_notebook.py`
  - `/api/v1/research-notebook/status`
  - `/api/v1/research-notebook/sync-out`
  - `/api/v1/research-notebook/ask`
- User/session notebook mapping stored in user storage JSON:
  - `integrations/research-notebook.json`
- Evidence Notebook UI in the Data page with first-party terminology.
- Removed the stale frontend NotebookLM client file; legacy backend NotebookLM route remains intact but no longer has the Data page UI path.
- Added tests:
  - `backend/tests/test_research_notebook.py`
  - `frontend/tests/evidence-notebook-copy.test.mjs`

Verification run by TBA-One-PA:

```bash
cd /Users/matejhavlin/github/applications/advisory-council-app/backend
./.venv/bin/python -m pytest -q tests/test_research_notebook.py
python -m py_compile app/services/research_notebook.py app/routers/research_notebook.py app/main.py app/config.py

cd /Users/matejhavlin/github/applications/advisory-council-app/frontend
npm run test:evidence-notebook-copy
npm run check
npm run build

cd /Users/matejhavlin/github/applications/advisory-council-app
docker compose config --quiet
git diff --check
```

Results:

- Backend focused tests: `3 passed`.
- Python compile: passed.
- Frontend Evidence Notebook copy regression: passed.
- Svelte check: `0 errors and 0 warnings`.
- Frontend build: passed.
- Docker Compose config validation: passed.
- Git whitespace check: passed.

Next step before production deploy:

- Set real `COUNCIL_OPEN_NOTEBOOK_PASSWORD` and `COUNCIL_OPEN_NOTEBOOK_ENCRYPTION_KEY` in dev-2026 CouncilNow env.
- Deploy/restart intentionally, then smoke `/api/v1/research-notebook/status` and the Data page Evidence Notebook flow.

## Brain alternatives check — 2026-04-26 13:55 CEST

Question: is Open Notebook the best open-source brain/NotebookLM replacement, compared with Open Brain / Karpathy-style wiki / GBrain / similar systems?

Verdict by use case:

1. **CouncilNow customer-facing Evidence Notebook:** Open Notebook remains the best starting choice.
   - It is directly NotebookLM-shaped: notebooks, sources, notes, search, ask/chat, podcasts, REST API, self-hosting.
   - It is MIT licensed and active (`lfnovo/open-notebook`, ~22.7k stars observed via GitHub CLI).
   - Its weakness is multi-tenant auth, which CouncilNow is already solving by keeping it internal behind CouncilNow backend permissions.

2. **Internal agent/company brain:** GBrain is stronger than Open Notebook.
   - Garry Tan's `gbrain` is built for agent memory, typed knowledge graphs, entity timelines, hybrid search, code graph retrieval, skills, cron/maintenance, and minion jobs.
   - It is not the best drop-in NotebookLM replacement UI for CouncilNow customers; it is better as TBA/OpenClaw/AgentJack internal memory infrastructure.

3. **Personal/wiki-style brain:** Karpathy LLM Wiki pattern / `NicholasSpisak/second-brain` is cleanest.
   - Best for LLM-maintained Obsidian/wiki knowledge pages.
   - Not enough as a SaaS Evidence Notebook engine: no product UI/API/tenant layer comparable to Open Notebook.

4. **Open Brain / OB1:** promising shared-memory infrastructure, not the safest CouncilNow product engine yet.
   - `NateBJones-Projects/OB1` positions itself as one database + AI gateway + chat channel for persistent memory across tools.
   - Interesting for personal/operator brain architecture; less directly NotebookLM-shaped than Open Notebook and license/product maturity needs deeper vetting.

5. **SurfSense:** strongest challenger for team NotebookLM replacement, but higher product risk today.
   - `MODSetter/SurfSense` is explicitly team-oriented, privacy-focused, with many connectors and multiplayer claims.
   - If CouncilNow later needs collaborative enterprise notebooks/connectors, re-evaluate SurfSense. Today, Open Notebook is safer because the current MVP only needs internal engine + REST API + source-grounded notebook semantics.

6. **NotebookLlama / InsightsLM / AnythingLLM / Onyx / Khoj / Open WebUI:** useful but not the best CouncilNow fit.
   - NotebookLlama is LlamaCloud-backed and more demo/reference than self-owned CouncilNow engine.
   - InsightsLM is more template/workflow stack (Supabase/N8N/React).
   - AnythingLLM/Open WebUI/Onyx/Khoj are broader chat/RAG/enterprise-search systems, not as close to NotebookLM-style evidence notebook UX.

Decision: keep the current implementation path. Use **Open Notebook for CouncilNow Evidence Notebook**, **GBrain for internal agent brain**, and **Karpathy/second-brain pattern for personal/wiki-style knowledge compilation**. Do not try to make one brain solve all three jobs.
