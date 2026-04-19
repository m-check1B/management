# AgentJack Harness Analysis: OpenClaw vs GoClaw

_Created: 2026-04-19 07:17 by Axis_

## TL;DR

**GoClaw is the better harness for AgentJack as a B2B product.** OpenClaw is our workbench (internal). GoClaw has what AgentJack needs for multi-tenant B2B built-in. But we don't adopt it blindly — we cherry-pick the architecture and keep AgentJack's unique value.

## Comparison

### What AgentJack Needs (B2B Product)
| Requirement | OpenClaw | GoClaw | AgentJack Has |
|-------------|----------|--------|---------------|
| Multi-tenant | ❌ File-based | ✅ PostgreSQL per-user | Partial (tenants.py added) |
| Encrypted secrets | ❌ Env vars | ✅ AES-256-GCM in DB | ❌ |
| Agent teams/orchestration | ❌ Basic subagents | ✅ Task boards + mailbox | ✅ Scheduler + evaluator |
| MCP integration | ❌ Uses ACP | ✅ stdio/SSE/streamable | ❌ |
| Knowledge graph | ❌ | ✅ LLM extraction + traversal | Partial (brain) |
| Skill system | ✅ Embeddings/semantic | ✅ BM25 + pgvector hybrid | ❌ |
| Lane-based scheduler | ✅ | ✅ (main/subagent/team/cron) | ✅ Built last night |
| Production security | ✅ SSRF/injection | ✅ 5-layer + rate limit + sandbox | Partial |
| Single binary deploy | ❌ Node.js runtime | ✅ ~25MB Go binary, <1s startup | ❌ Python + Node |
| Docker image | — | ✅ ~50MB Alpine | ❌ |
| Web dashboard | ✅ Control UI | ✅ Built-in React SPA | ✅ ag-webui (SvelteKit) |
| RBAC | ❌ | ✅ Full | ❌ |
| Per-user workspaces | ✅ File-based | ✅ PostgreSQL-backed | Partial |
| Desktop app | ✅ macOS/iOS/Android | ✅ Wails (macOS/Windows) | ❌ |
| Observability | ✅ Opt-in OTel | ✅ OTLP + built-in tracing | ❌ |
| Prompt caching | ❌ | ✅ Anthropic + OpenAI-compat | ❌ |

### What AgentJack Has That Neither Has
1. **Advisory Council integration** — decision engine as first-class product feature
2. **Brain product** — multi-perspective analysis
3. **Courseroom/Classroom** — training vertical
4. **OpenCode bridge** — wired to our coding agents
5. **CouncilNow revenue path** — actual product with customers
6. **The orchestrator vision** — agent core → apps → surpass everything

### What GoClaw Has That AgentJack Needs
1. **Multi-tenant PostgreSQL** — essential for B2B
2. **Encrypted API keys in DB** — not in env vars
3. **Agent teams with task boards** — real orchestration
4. **MCP support** — 2026 standard, we need it
5. **Knowledge graph** — our brain should use this
6. **Single binary deployment** — B2B customers want `docker-compose up`
7. **5-layer security** — enterprise-grade
8. **~25MB binary** vs our Python+Node stack

## Three Options

### Option A: Keep Building on AgentJack (Current Path)
- Continue Python + Node.js stack
- Build multi-tenant, MCP, knowledge graph ourselves
- **Pros:** Full control, unique architecture, already started
- **Cons:** Slow, reinventing what GoClaw already built, deployment complexity
- **Timeline to B2B-ready:** 3-6 months

### Option B: Fork GoClaw, Port AgentJack's Soul
- Fork GoClaw as the runtime harness
- Port AgentJack's products (Brain, Council, Courseroom) as GoClaw apps
- Port our scheduler/evaluator logic
- **Pros:** Get multi-tenant + security + deploy for free, focus on product
- **Cons:** Go codebase (new language), license is CC BY-NC 4.0 (non-commercial!), major rewrite
- **Timeline to B2B-ready:** 2-4 months
- **⚠️ LICENSE BLOCKER:** GoClaw is CC BY-NC 4.0 — **we CANNOT use it commercially without a commercial license.** This kills Option B unless we negotiate with the author.

### Option C: Hybrid — GoClaw Architecture, AgentJack Implementation
- Study GoClaw's architecture (multi-tenant, security layers, knowledge graph)
- Reimplement the patterns in our Python/Node stack
- Keep AgentJack's unique products and vision
- **Pros:** Best ideas from GoClaw, full control, no license issues, our stack
- **Cons:** Still building, but with a proven blueprint
- **Timeline to B2B-ready:** 2-3 months

## Recommendation

**Option C — Hybrid.** Here's why:

1. **License kills Option B.** CC BY-NC 4.0 means we can't sell AgentJack if it's based on GoClaw code. Dealbreaker.
2. **Option A is too slow.** We'd reinvent multi-tenant PostgreSQL, MCP, knowledge graph, encrypted secrets, all while GoClaw already proved the patterns work.
3. **Option C gives us the blueprint without the license trap.** Study GoClaw's `internal/` packages — their multi-tenant store, 5-layer security, agent loop, knowledge graph, MCP integration — and implement equivalent patterns in our Python stack.

### What to Port from GoClaw (Patterns, Not Code)
| GoClaw Pattern | AgentJack Implementation |
|----------------|-------------------------|
| `internal/store/` multi-tenant | PostgreSQL with tenant isolation in our Python backend |
| `internal/permissions/` 5-layer | Permission middleware in FastAPI |
| `internal/agent/loop*.go` agent loop | Our scheduler + evaluator already partially built |
| `internal/knowledgegraph/` | Our brain product, enhanced with pgvector |
| `internal/mcp/` | MCP client in Python (already have mcporter) |
| `internal/crypto/` encrypted secrets | AES-256-GCM for API keys in DB |
| `internal/webui/` dashboard | Our ag-webui SvelteKit (already better) |

### What NOT to Port
- Go language / single binary (we're Python + Node, that's fine)
- Their React UI (we have SvelteKit)
- Their messaging channels (we have our own via OpenClaw internally)
- Their skill search (BM25 — our gbrain hybrid search is better)

## Next Steps

1. Read GoClaw's `internal/store/` for multi-tenant schema patterns
2. Read GoClaw's `internal/permissions/` for security layer design
3. Read GoClaw's `internal/agent/loop.go` for the agent loop architecture
4. Design AgentJack's PostgreSQL schema based on these patterns
5. Implement multi-tenant store in AgentJack's Python backend
6. Wire existing scheduler/evaluator to the new store

## License Note

**GoClaw is CC BY-NC 4.0.** We can study the architecture and implement our own version. We cannot copy code or use it commercially without a license from nextlevelbuilder. This is standard practice — learn from open source, build your own implementation.
