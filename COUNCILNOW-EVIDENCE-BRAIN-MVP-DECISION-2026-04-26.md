# CouncilNow Evidence Brain MVP Decision — 2026-04-26

## Decision

Implement Evidence Brain MVP **directly inside the CouncilNow backend**.

Do not start with a separate service.
Do not embed Synthadoc.
Keep Open Notebook as the already-deployed Evidence Notebook workbench sidecar.

## Why

1. **Permission boundary is everything.** CouncilNow already owns auth, tenancy, billing, storage, sessions, and audit context. Evidence Brain must inherit that boundary by default.
2. **Council runs need tight integration.** The first valuable output is an Evidence Packet injected into advisor deliberation, not a standalone evidence API.
3. **Fastest path to revenue-quality product.** A service boundary adds deploy, auth, network, and observability work before the product proves the evidence loop.
4. **Synthadoc benchmark showed the right pattern but wrong source of truth.** It compiled a useful wiki and query answer, but failed to flag obvious contradictions in lint and only cited wiki pages, not raw source spans.
5. **AGPL risk stays outside product.** Synthadoc remains a reference/benchmark until legal review approves anything stronger.

## Architecture

```text
CouncilNow backend
  ├─ evidence_raw_sources
  ├─ evidence_source_chunks
  ├─ evidence_claims
  ├─ evidence_citations
  ├─ evidence_contradictions
  ├─ evidence_entities
  ├─ evidence_relationships
  ├─ evidence_timeline_events
  ├─ evidence_packets
  └─ evidence_audit_events
```

Initial module layout:

```text
backend/app/models/evidence.py
backend/app/services/evidence_ingest.py
backend/app/services/evidence_claims.py
backend/app/services/evidence_contradictions.py
backend/app/services/evidence_packets.py
backend/app/routers/evidence.py
backend/app/migrations/*evidence_brain*
```

## MVP slice

Build the minimum backend slice that beats the Synthadoc benchmark:

1. Upload/import md/txt/csv/json source.
2. Store immutable raw source with SHA-256.
3. Chunk source into cited spans/rows.
4. Extract typed claims.
5. Detect deterministic contradictions:
   - numeric mismatch: EUR 500,000 vs EUR 650,000,
   - requirement/status conflict: traffic mitigation required vs optional/deferred.
6. Compile Evidence Packet as Markdown + JSON.
7. Attach packet to a CouncilNow session.
8. Save final council report as a Decision artifact with citations.

## Acceptance test

Use the exact tiny benchmark corpus:

```text
agenda-minutes.md
budget.csv
contradiction.md
```

Pass condition:

- Evidence Brain creates typed claims from all three files.
- It detects both expected contradictions.
- It emits source-span citations with filename + location + hash.
- A council run receives an evidence packet that lists unresolved contradictions.
- Final report reasons over contradictions instead of blending them.

## Future split conditions

Only split Evidence Brain into a separate service if one of these becomes true:

- multiple products need the same evidence runtime,
- ingest load threatens CouncilNow API stability,
- enterprise deployment needs separate data plane,
- legal review approves an isolated unmodified third-party service boundary.

Until then: direct backend wins.
