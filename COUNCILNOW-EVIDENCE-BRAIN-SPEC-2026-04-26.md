# CouncilNow Evidence Brain Spec — 2026-04-26

## Decision

Build **CouncilNow Evidence Brain** as a CouncilNow-owned backend capability, not as an Open Notebook fork and not as direct embedded Synthadoc code.

Open Notebook remains the fast **Evidence Notebook** workbench. Evidence Brain is the durable, tenant-safe, cited institutional memory that improves council-session quality.

## Product contract

CouncilNow should be able to answer and run councils from evidence like this:

1. A user uploads, imports, pins, or syncs raw material.
2. CouncilNow stores immutable raw sources with hashes and ownership.
3. CouncilNow extracts cited claims, entities, relationships, and timeline events.
4. CouncilNow detects contradictions before a council run.
5. CouncilNow builds an evidence packet for a session.
6. Advisors deliberate over structured evidence, not loose document chat.
7. The final report becomes a cited decision artifact and feeds future memory.

## Naming

- **Raw Source** — original uploaded/imported source; immutable.
- **Evidence Brain** — CouncilNow-owned compiled evidence system.
- **Evidence Graph** — typed entities/relationships/timeline/claims.
- **Evidence Notebook** — Open Notebook-backed workbench for quick source Q&A.
- **Evidence Packet** — scoped evidence bundle injected into a council session.

## Core primitives

### RawSource

Immutable original source record.

Required fields:

```text
id
org_id / tenant_id
user_id / uploaded_by
source_type: upload | pinned_file | github_snapshot | url | council_session | manual_note | api_import
mime_type
original_filename
storage_uri
sha256
byte_size
language
created_at
retention_policy
legal_hold_state
deleted_at nullable
metadata_json
```

Rules:

- Raw source bytes are never mutated.
- Re-ingest creates new derived artifacts, not a new raw source, when hash matches.
- Deletion policy must support hard delete, export, and legal hold.

### SourceChunk

Addressable source segment used for citation and extraction.

Fields:

```text
id
raw_source_id
org_id / tenant_id
chunk_index
text
text_sha256
location_kind: page | row | sheet | heading | paragraph | timestamp | byte_range
location_json
char_start
char_end
token_count
embedding_ref nullable
created_at
```

Rules:

- Chunk boundaries must preserve citation location.
- Spreadsheet rows and PDF pages need first-class locations, not generic text blobs.

### Claim

Atomic factual assertion extracted from source chunks.

Fields:

```text
id
org_id / tenant_id
raw_source_id
source_chunk_id
claim_text
normalized_subject
normalized_predicate
normalized_object
claim_type: fact | number | date | decision | requirement | risk | commitment | forecast | opinion
polarity: asserts | denies | qualifies
value_text nullable
value_number nullable
value_unit nullable
value_date nullable
valid_from nullable
valid_to nullable
extracted_by_model
extracted_by_provider
extraction_prompt_version
confidence
human_review_state: unreviewed | accepted | rejected | edited
contradiction_status: none | potential | contradicted | resolved
created_at
updated_at
```

Rules:

- Claims are the unit of truth, contradiction, citation, and council grounding.
- Free-text summaries are secondary; structured claim rows are primary.

### EvidenceCitation

Stable citation from claim/report text back to raw source span.

Fields:

```text
id
org_id / tenant_id
claim_id nullable
source_chunk_id
raw_source_id
citation_label
citation_text
quote
location_json
source_sha256
created_at
```

Citation format:

```text
[Source: <filename>, <page/row/section>, sha256:<first12>]
```

UI format can be friendlier, but backend must store source hash + exact location.

### Contradiction

Potential or confirmed conflict between two or more claims.

Fields:

```text
id
org_id / tenant_id
claim_a_id
claim_b_id
contradiction_type: numeric_mismatch | date_mismatch | status_conflict | requirement_conflict | entity_conflict | semantic_conflict
summary
severity: low | medium | high | blocker
confidence
status: open | accepted | rejected | resolved | superseded
resolution_note nullable
resolved_by nullable
resolved_at nullable
detected_by_model
detected_by_provider
detection_prompt_version
created_at
```

Rules:

- Numeric/date/entity mismatches should be deterministic before LLM review.
- LLM contradiction detection is allowed for semantic conflicts, but never final authority without human/system review state.

### Entity

Canonical people, organizations, projects, places, topics, policies, vendors, assets.

Fields:

```text
id
org_id / tenant_id
entity_type: person | organization | project | place | policy | vendor | asset | topic | metric
canonical_name
aliases_json
external_ids_json
confidence
created_from_claim_id nullable
human_review_state
created_at
updated_at
```

### Relationship

Typed edge between entities or artifacts.

Fields:

```text
id
org_id / tenant_id
subject_entity_id
predicate: works_at | member_of | vendor_for | owns | affects | supports | refutes | produced | references | replaces | depends_on
object_entity_id nullable
object_artifact_type nullable
object_artifact_id nullable
claim_id nullable
confidence
valid_from nullable
valid_to nullable
created_at
```

### TimelineEvent

Time-addressed evidence or decision event.

Fields:

```text
id
org_id / tenant_id
event_type: meeting | decision | contract | deadline | payment | vote | publication | incident | milestone
title
description
event_date
source_claim_id nullable
source_citation_id nullable
entity_ids_json
confidence
created_at
```

### Decision

CouncilNow decision artifact, including council outputs.

Fields:

```text
id
org_id / tenant_id
user_id
session_id nullable
title
question
summary
recommendation
confidence_score
advisor_vote_json
source_packet_id nullable
report_uri nullable
status: draft | final | superseded | archived
created_at
updated_at
```

### CouncilSessionEvidencePacket

Versioned evidence bundle injected into a council session.

Fields:

```text
id
org_id / tenant_id
session_id
topic
query_text
included_raw_source_ids_json
included_claim_ids_json
included_entity_ids_json
included_relationship_ids_json
included_timeline_event_ids_json
open_contradiction_ids_json
packet_markdown
packet_json
compiled_by_model nullable
compiled_by_provider nullable
compile_prompt_version
created_at
```

Rules:

- Packet is immutable after council run starts.
- Packet must include unresolved contradictions, not hide them.
- Advisors receive evidence packet + citations, not unrestricted source dump.

### AuditEvent

Append-only event trail.

Fields:

```text
id
org_id / tenant_id
actor_type: user | system | agent | api_key
actor_id
artifact_type
artifact_id
event_type
before_json nullable
after_json nullable
request_id nullable
ip_hash nullable
created_at
```

Events:

```text
raw_source.uploaded
raw_source.deleted
source.chunked
claim.extracted
claim.reviewed
contradiction.detected
contradiction.resolved
entity.merged
evidence_packet.compiled
council_session.started_with_evidence
decision.finalized
export.requested
```

## Backend architecture

Recommended MVP is **direct CouncilNow backend module**, not a separate service yet.

Why direct first:

- Reuses existing auth, tenancy, billing, storage, and session models.
- Faster to ship and easier to keep permission boundaries correct.
- Evidence packets need tight integration with council runs.
- Avoids premature service boundary + operational overhead.

Where to place it:

```text
backend/app/models/evidence.py          # SQLAlchemy models
backend/app/services/evidence_ingest.py # raw source -> chunks
backend/app/services/evidence_claims.py # chunks -> claims
backend/app/services/evidence_graph.py  # entities/relationships/timeline
backend/app/services/evidence_packets.py
backend/app/routers/evidence.py
backend/app/migrations/*evidence_brain*
```

When to split into isolated service:

- Multiple products need the same Evidence Brain runtime.
- Heavy background ingest saturates CouncilNow API workers.
- Customer deployment requires separate data plane.
- Legal review approves a strict unmodified third-party service boundary for AGPL tools.

## Storage model

Use existing CouncilNow Postgres for metadata and graph v1.

Initial tables:

```text
evidence_raw_sources
evidence_source_chunks
evidence_claims
evidence_citations
evidence_contradictions
evidence_entities
evidence_relationships
evidence_timeline_events
evidence_packets
evidence_decisions
evidence_audit_events
```

Raw bytes:

- local/dev: existing user storage path
- production: current CouncilNow storage abstraction; future S3-compatible bucket if needed

Search:

- MVP: Postgres full-text + deterministic filters
- Later: pgvector or GBrain-style hybrid retrieval
- Graph MVP: typed SQL edges; no Neo4j requirement

## Ingest pipeline

```text
RawSource
  -> SourceChunker
  -> ClaimExtractor
  -> EntityResolver
  -> RelationshipExtractor
  -> TimelineExtractor
  -> ContradictionDetector
  -> AuditEvent append
```

MVP extraction stages:

1. Deterministic text extraction for txt/md/csv/json.
2. PDF/docx/xlsx extraction using existing Python libraries if already available; otherwise add smallest stable dependency.
3. LLM claim extraction through existing savings-router/model policy.
4. Deterministic contradiction checks for numeric/date/status conflicts.
5. LLM semantic contradiction check only after deterministic pass.

## Contradiction rules

Start with high-value deterministic detectors:

- Same subject + predicate + numeric value mismatch above tolerance.
- Same subject + status conflict (`approved` vs `rejected`, `required` vs `optional`).
- Same entity/date conflict.
- Same decision with incompatible caps/amounts.

Example from benchmark:

```text
Claim A: Phase 1 public contribution cap is EUR 500,000.
Claim B: Phase 1 requires public contribution of EUR 650,000 before work can begin.
Contradiction: numeric_mismatch, severity=high.
```

## Evidence packet format

Packet should be both Markdown and JSON.

Markdown skeleton:

```markdown
# Evidence Packet: <topic>

## Key claims
n. <claim> [Source: ...]

## Open contradictions
n. <summary>
   - Claim A ... [Source]
   - Claim B ... [Source]

## Timeline
- YYYY-MM-DD — event [Source]

## Relevant entities
- Entity — why relevant

## Sources included
- filename, hash, created_at
```

JSON must include IDs so the final report can cite exact records.

## API MVP

```text
POST /api/v1/evidence/sources
GET  /api/v1/evidence/sources
GET  /api/v1/evidence/sources/{id}
POST /api/v1/evidence/sources/{id}/ingest
GET  /api/v1/evidence/claims?source_id=&entity_id=&status=
GET  /api/v1/evidence/contradictions?status=open
POST /api/v1/evidence/contradictions/{id}/resolve
POST /api/v1/evidence/packets
GET  /api/v1/evidence/packets/{id}
POST /api/v1/sessions/{id}/evidence-packet
```

## UI MVP

Do not build full wiki UI first.

First native surfaces:

1. Evidence Inbox — raw sources + ingest status.
2. Contradictions Queue — open conflicts before session run.
3. Evidence Packet Preview — what advisors will receive.
4. Cited Report Evidence tab — claims/citations behind final recommendation.

Evidence Notebook remains available in Data page for exploratory Q&A.

## Permissions and tenancy

Non-negotiables:

- Every row has org/user ownership.
- API filters by current auth context before any model call.
- Open Notebook native API is never customer-facing.
- Evidence packets are compiled server-side only from authorized artifacts.
- Export/delete flows traverse raw source, chunks, claims, citations, graph edges, packets, and audit summaries.

## Build order

### Slice 1 — metadata + deterministic extraction

- Tables: RawSource, SourceChunk, Claim, Citation, AuditEvent.
- Upload md/txt/csv/json.
- Deterministic chunking + source hashes.
- Manual or simple model-assisted claim extraction.

### Slice 2 — contradictions + packet

- Add Contradiction table and deterministic numeric/status conflict detector.
- Build Evidence Packet compiler.
- Attach packet to council session prompt.

### Slice 3 — graph + timeline

- Add Entity, Relationship, TimelineEvent.
- Entity resolver and relationship extractor.
- Timeline included in packet.

### Slice 4 — UI productization

- Evidence Inbox.
- Contradictions Queue.
- Packet Preview.

## Acceptance gate for MVP

A test council session can ingest the three-source benchmark corpus and produce:

- two conflicting claims about Phase 1 public contribution (`500,000` vs `650,000`),
- one requirement/status conflict about traffic mitigation (`required plan` vs `optional/deferred`),
- an evidence packet listing both conflicts with citations,
- a council report that explicitly reasons over the conflicts instead of blending them.

## Explicit non-goals for MVP

- No public Open Notebook route.
- No direct Synthadoc code embedding.
- No AGPL-derived implementation copy.
- No full Obsidian/wiki UI.
- No cross-tenant graph.
- No autonomous deletion of source material.

## Legal/licensing note

Synthadoc is AGPL-3.0. For now:

- Allowed: benchmark, read behavior, extract product patterns, write independent implementation.
- Avoid: copying code, modifying and serving it, linking it as a product dependency, or embedding it in CouncilNow without legal review.

## Recommendation

Implement Phase 4 MVP **directly in CouncilNow backend** as an evidence module with SQL graph tables. Keep Open Notebook as the optional workbench sidecar. Treat Synthadoc as benchmark/reference only until legal review says otherwise.
