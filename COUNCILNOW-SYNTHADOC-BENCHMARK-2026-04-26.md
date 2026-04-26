# CouncilNow Synthadoc Benchmark — 2026-04-26

## Purpose

Benchmark Synthadoc as a **reference/blueprint**, not as a CouncilNow product dependency, for the CouncilNow Evidence Brain plan.

License guardrail: Synthadoc is AGPL-3.0. This run evaluated behavior only. No Synthadoc code was copied into CouncilNow.

## Setup

Repo:

```text
/Users/matejhavlin/github/synthadoc
```

Command path:

```text
uv run synthadoc ...
```

Wiki:

```text
/tmp/council-evidence-bench
```

Install command:

```bash
uv run synthadoc install council-evidence-bench --target /tmp --domain "Council evidence benchmark: decisions, budgets, contradictions, citations"
```

Server command:

```bash
uv run synthadoc serve -w council-evidence-bench --verbose
```

Provider observed:

```text
default=gemini/gemini-2.5-flash-lite
```

Tavily was missing, but irrelevant because this benchmark used local files only.

## Corpus

Created three tiny local sources:

```text
/tmp/synthadoc-bench-sources/agenda-minutes.md
/tmp/synthadoc-bench-sources/budget.csv
/tmp/synthadoc-bench-sources/contradiction.md
```

### agenda-minutes.md

- Riverfront Redevelopment Phase 1 recommended for approval.
- Condition: contractor provides a traffic mitigation plan.
- Maximum public contribution: EUR 500,000.

### budget.csv

- `public_contribution = 500000`
- `traffic_mitigation = 45000`
- `community_outreach = 12000`

### contradiction.md

- Contractor says Phase 1 requires EUR 650,000 before work can begin.
- Contractor says traffic mitigation is optional and can be deferred to Phase 2.

Expected conflicts:

1. EUR 500,000 cap vs EUR 650,000 required.
2. Traffic mitigation required plan vs optional/deferred.

## Commands run

```bash
uv run synthadoc ingest --batch /tmp/synthadoc-bench-sources -w council-evidence-bench
uv run synthadoc jobs list -w council-evidence-bench
uv run synthadoc audit history -w council-evidence-bench --json
uv run synthadoc lint run --scope contradictions -w council-evidence-bench
uv run synthadoc lint report -w council-evidence-bench
uv run synthadoc query "What contradictions exist in the Riverfront Redevelopment evidence about public contribution and traffic mitigation? Cite the sources." -w council-evidence-bench --save
uv run synthadoc audit cost -w council-evidence-bench --json
```

## Results

### Jobs

```text
6d8ff135  completed     ingest    budget.csv
5f6b3eb4  completed     ingest    contradiction.md
19ff1feb  completed     ingest    agenda-minutes.md
52c91107  completed     lint      contradictions
```

### Audit history

```json
[
  {
    "source_path": "/private/tmp/synthadoc-bench-sources/budget.csv",
    "wiki_page": "index",
    "tokens": 1345,
    "cost_usd": 0.0,
    "ingested_at": "2026-04-26T12:19:29.427657+00:00"
  },
  {
    "source_path": "/private/tmp/synthadoc-bench-sources/contradiction.md",
    "wiki_page": "riverfront-redevelopment-project",
    "tokens": 1139,
    "cost_usd": 0.0,
    "ingested_at": "2026-04-26T12:19:33.684368+00:00"
  },
  {
    "source_path": "/private/tmp/synthadoc-bench-sources/agenda-minutes.md",
    "wiki_page": "riverfront-redevelopment-project",
    "tokens": 1479,
    "cost_usd": 0.0,
    "ingested_at": "2026-04-26T12:19:38.063073+00:00"
  }
]
```

Cost report:

```json
{
  "total_tokens": 4870,
  "total_cost_usd": 0.0001002,
  "daily": [
    {
      "day": "2026-04-26",
      "cost_usd": 0.0001002
    }
  ]
}
```

### Wiki artifacts created

```text
/tmp/council-evidence-bench/wiki/dashboard.md
/tmp/council-evidence-bench/wiki/index.md
/tmp/council-evidence-bench/wiki/purpose.md
/tmp/council-evidence-bench/wiki/riverfront-redevelopment-project.md
```

Synthadoc merged the contractor brief and minutes into one page:

```text
riverfront-redevelopment-project.md
```

The page contained both conflicting facts:

- contractor brief: EUR 650,000 required; traffic mitigation optional/deferred.
- phase approval conditions: traffic mitigation plan required; max public contribution EUR 500,000.

### Lint result

```text
0 contradiction(s), 1 orphan(s) found.
```

Important: the contradiction lint **did not flag** the two obvious conflicts, even though the query could surface them later.

### Query result

The query correctly described both conflicts in natural language:

- EUR 650,000 required before commencement vs EUR 500,000 maximum public contribution.
- Traffic mitigation optional/deferred vs Phase 1 approval contingent on a traffic mitigation plan.

But citations were wiki-page-level, not raw-source-span-level:

```text
Sources: [[purpose]], [[dashboard]], [[index]], [[riverfront-redevelopment-project]]
```

This is useful for a wiki, but not enough for CouncilNow's audit/citation requirements.

## Evaluation

| Capability | Result | CouncilNow implication |
|---|---:|---|
| Local wiki setup | Passed | Good blueprint for raw-source → compiled artifact discipline |
| Batch ingest | Passed | Queue/audit model worth copying conceptually |
| Markdown artifact | Passed | Human-readable compiled evidence is valuable |
| Audit history | Passed | Source hash/audit DB concept maps directly to Evidence Brain |
| Cost reporting | Passed | Useful but CouncilNow should reuse existing model-routing/cost policy |
| Contradiction lint | Failed for benchmark | CouncilNow needs deterministic claim-level contradiction detection, not page-status lint only |
| Query-time contradiction explanation | Passed | Useful fallback, but not enough as source of truth |
| Raw-source citations | Weak | CouncilNow must store chunk/span citations itself |
| Tenancy/auth model | Not evaluated | CouncilNow must own this boundary entirely |

## Patterns worth stealing

1. **Ingest-time compilation**
   - Do not wait until query time to synthesize evidence.
   - Create persistent artifacts that humans and agents can inspect.

2. **Audit DB / source hash discipline**
   - Every ingest should record source hash, source path/URI, timestamp, tokens, and cost.
   - CouncilNow should make this tenant-aware and exportable.

3. **Plain Markdown packet/artifact**
   - CouncilNow evidence packets should have Markdown and JSON forms.
   - Markdown is excellent for preview, export, and debugging.

4. **Queue-based ingest**
   - Jobs should be resumable and inspectable.
   - Failed ingest should not poison the whole product.

5. **Orphan/quality lint idea**
   - CouncilNow can lint evidence for uncited claims, orphan entities, stale sources, and unresolved contradictions.

## Patterns not to adopt directly

1. **AGPL product dependency**
   - Do not embed/modify/serve Synthadoc as a CouncilNow dependency without legal review.

2. **Page-level contradiction status as source of truth**
   - The benchmark failed to flag obvious conflicts with lint.
   - CouncilNow needs typed claim rows and deterministic conflict checks.

3. **Wiki-page citations only**
   - CouncilNow needs exact source span/page/row citations and source hashes.

4. **Generic wiki schema**
   - CouncilNow needs decision-specific primitives: RawSource, Claim, Contradiction, Decision, EvidencePacket, AuditEvent.

## Decision

Use Synthadoc as **reference and benchmark only** for now.

Do **not** embed Synthadoc in CouncilNow MVP.

Build CouncilNow Evidence Brain directly inside the CouncilNow backend, with:

- immutable raw sources,
- typed claims,
- source-span citations,
- deterministic contradiction detection,
- evidence packet compiler,
- council-run integration.

## Next implementation target

Create a first CouncilNow backend slice that can pass the same benchmark better than Synthadoc:

1. ingest the three sample files,
2. extract typed claims,
3. detect both conflicts deterministically,
4. compile an Evidence Packet with exact citations,
5. attach the packet to a council run.
