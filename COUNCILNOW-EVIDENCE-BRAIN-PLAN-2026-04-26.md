# CouncilNow Evidence Brain Plan — 2026-04-26

## Forced decision

CouncilNow should not become “chat with documents.” It should become a defensible decision/evidence system.

Final architecture:

```text
CouncilNow Evidence Brain
= Synthadoc-style evidence compiler
+ GBrain-style typed graph / agent memory
+ Karpathy raw_sources → wiki discipline
+ CouncilNow-owned permissions, audit, tenancy, citations
+ Open Notebook-style Evidence Notebook workbench
```

## Role split

| Layer | Role | Decision |
|---|---|---|
| CouncilNow | Product, tenancy, permissions, audit, billing, UX | Own completely |
| Open Notebook | NotebookLM replacement workbench/UI/API | Use now as internal sidecar |
| Synthadoc | Evidence compiler blueprint | Study/prototype; do not embed modified AGPL code without legal review |
| GBrain | Typed graph/agent memory inspiration | Use concepts/patterns; permissive MIT safer for direct pattern adoption |
| Karpathy LLM Wiki | Design philosophy | Raw immutable sources → compiled wiki → schema discipline |
| OB1/Open Brain | Personal AI memory bus | Do not make core CouncilNow brain |

## Current state

Already implemented and pushed:

- CouncilNow commit: `8d134ab Add CouncilNow Evidence Notebook sidecar`
- Adds internal Open Notebook sidecar services:
  - `open-notebook-surrealdb`
  - `open-notebook`
- Adds backend proxy endpoints:
  - `GET /api/v1/research-notebook/status`
  - `POST /api/v1/research-notebook/sync-out`
  - `POST /api/v1/research-notebook/ask`
- Replaces user-facing NotebookLM bridge in Data page with **Evidence Notebook** UI.
- No production deploy yet.

Management evidence:

- `COUNCILNOW-OPEN-NOTEBOOK-EVAL-2026-04-26.md`
- Latest architecture clarification commit: `1826e70 Clarify CouncilNow Evidence Brain architecture`

Synthadoc cloned locally:

- `/Users/matejhavlin/github/synthadoc`
- Upstream: `axoviq-ai/synthadoc`
- License: AGPL-3.0
- Inspected commit: `a91a8ba`

## Phase 0 — Freeze the mental model

Do not call Open Notebook “the brain.”

Use names precisely:

- **Evidence Notebook** = user-facing workbench.
- **Evidence Brain** = compiled, cited, auditable institutional memory.
- **Evidence Graph** = entities/relationships/timelines/claims.
- **Raw Sources** = immutable uploaded/imported originals.

## Phase 1 — Deploy Open Notebook sidecar safely

Goal: remove Google NotebookLM from the user workflow without overbuilding.

Tasks:

1. Set real production env values on dev-2026:
   - `COUNCIL_OPEN_NOTEBOOK_PASSWORD`
   - `COUNCIL_OPEN_NOTEBOOK_ENCRYPTION_KEY`
   - confirm `COUNCIL_OPEN_NOTEBOOK_URL=http://open-notebook:5055`
2. Deploy CouncilNow intentionally from commit `8d134ab` or newer.
3. Verify sidecar is internal-only:
   - no public Traefik route
   - ports bound to localhost/internal network only
   - no direct customer access to Open Notebook native UI/API
4. Smoke:
   - `/api/v1/research-notebook/status`
   - Data page Evidence Notebook status card
   - sync pinned file → source appears
   - ask scoped notebook → answer returns
5. Keep Google OAuth untouched.

Exit gate:

- User can sync CouncilNow evidence into internal Open Notebook and ask from CouncilNow UI.
- No NotebookLM auth JSON flow is visible.
- Raw Open Notebook is not public.

## Phase 2 — Evidence Brain design doc

Goal: define CouncilNow-owned brain schema before coding.

Design these primitives:

```text
RawSource
SourceChunk
Claim
EvidenceCitation
Contradiction
Entity
Relationship
TimelineEvent
Decision
CouncilSessionEvidencePacket
AuditEvent
```

Required properties:

- tenant/user/org ownership
- source hash
- source location/page/section
- extraction timestamp
- model/provider used
- confidence
- contradiction status
- human review state
- citation format
- deletion/export policy

Output:

- `COUNCILNOW-EVIDENCE-BRAIN-SPEC-2026-04-26.md`

## Phase 3 — Synthadoc benchmark/prototype, not product dependency

Goal: learn what to steal.

Tasks:

1. Run Synthadoc locally on a tiny sample corpus:
   - one council agenda/minutes PDF or markdown
   - one budget/spreadsheet-like source
   - one contradictory statement/source
2. Evaluate outputs:
   - citations
   - contradiction detection
   - generated wiki structure
   - audit DB contents
   - CLI/HTTP ergonomics
3. Document patterns worth copying:
   - ingest-time compilation
   - contradiction queue
   - orphan-page lint
   - source hash audit
   - Markdown artifact model
4. Document patterns not to adopt:
   - AGPL direct embedding risk
   - any weak auth/tenant assumptions

Exit gate:

- Decision whether to use Synthadoc only as reference or as isolated unmodified service after legal review.

## Phase 4 — CouncilNow Evidence Brain MVP

Goal: build our own minimal permissive evidence compiler inside CouncilNow.

Initial MVP behavior:

1. Ingest uploaded/pinned source.
2. Extract claims with citations.
3. Store immutable raw source + source hash.
4. Create entities and timeline events.
5. Detect direct contradictions against prior claims.
6. Build an evidence packet for a council session.
7. Feed evidence packet into council deliberation.
8. Save council report back as a cited decision artifact.

Do not start with full wiki UI. Start with backend artifacts and council-run quality.

Exit gate:

- A council run can cite evidence claims and surface contradictions instead of blending them.

## Phase 5 — GBrain-style graph upgrade

Goal: make CouncilNow remember civic/business reality over time.

Add typed graph edges:

```text
Person --works_at--> Organization
Person --member_of--> Council/Board
Organization --vendor_for--> Organization
Decision --affects--> Entity
Claim --supports/refutes--> Decision
Source --contains--> Claim
Meeting --produced--> Source
```

Add graph retrieval into council context:

- relevant people/orgs
- past decisions
- relationship conflicts
- timeline around topic
- unresolved contradictions

Exit gate:

- CouncilNow can answer: “what changed since the last decision on this topic?” with citations.

## Phase 6 — Productize the Evidence Notebook UX

Goal: make the brain visible and useful to customers.

UI surfaces:

- Evidence Inbox
- Source library
- Claims view
- Contradictions queue
- Entity timeline
- Evidence packet preview before council run
- Cited report export

Open Notebook may remain behind the scenes or be gradually replaced by native CouncilNow UI if needed.

## Risks

| Risk | Mitigation |
|---|---|
| AGPL contamination from Synthadoc | Do not modify/embed without legal review; use as blueprint/benchmark first |
| Overbuilding before revenue | Deploy Open Notebook workbench first, then brain MVP only around council quality |
| Confusing UI workbench with brain | Keep naming strict: Notebook vs Brain |
| Tenant leakage | CouncilNow backend remains sole permission boundary |
| Citation hallucination | Store source spans/hashes; require cited claim objects, not free-text only |
| Google dependency confusion | Google OAuth is allowed; Google NotebookLM workflow is not |

## Immediate next actions after compress

1. Deploy Evidence Notebook sidecar intentionally to dev-2026 after setting real env secrets.
2. Smoke the CouncilNow Data page Evidence Notebook flow.
3. Write Evidence Brain schema/spec.
4. Run a tiny Synthadoc benchmark corpus locally.
5. Decide whether Phase 4 MVP should be implemented directly in CouncilNow backend or as isolated service.

## One-line strategy

Ship Open Notebook as the fast Google NotebookLM replacement workbench, but build CouncilNow’s durable moat as a Synthadoc-style, GBrain-informed, CouncilNow-owned Evidence Brain.
