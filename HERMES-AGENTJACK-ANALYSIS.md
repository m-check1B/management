# Hermes v0.10.0 → AgentJack Analysis

**Date:** 2026-04-19
**Hermes version:** v0.10.0 (latest, updated from v0.6.0)
**AgentJack repo:** /Users/matejhavlin/github/agentjack

## Hermes Architecture (What They Built)

### Web Dashboard (v0.9.0)
- **Stack:** React + Vite + TypeScript + Tailwind + shadcn/ui
- **Backend:** FastAPI (Python) serving API + static frontend
- **Location:** `hermes_cli/web_server.py` (2320 lines) + `web/` (46 files)
- **Auth:** Session token injected into index.html, Bearer auth on API calls
- **Pages:** Status, Sessions, Analytics, Logs, Cron, Skills, Config, Keys, Plugins
- **Plugin system:** External plugins can register nav tabs + routes + API endpoints
- **API pattern:** RESTful `/api/status`, `/api/sessions`, `/api/config`, `/api/cron/jobs`, etc.
- **Port:** 9119 default

### Profiles / Multi-Tenant (v0.6.0)
- **Location:** `hermes_cli/profiles.py` (1085 lines)
- **Pattern:** Each profile = isolated HERMES_HOME directory
- **Structure:** `~/.hermes/profiles/<name>/` with own config.yaml, .env, memory, sessions, skills, cron, logs
- **Isolation:** Token-lock prevents credential collision between profiles
- **Access:** `hermes -p <name>` flag or wrapper script at `~/.local/bin/<name>`
- **Export/import:** Profile export as portable archive
- **"default" profile:** `~/.hermes` itself — zero migration needed

### Pluggable Memory Provider (v0.7.0)
- **Location:** `agent/memory_provider.py` (ABC) + `agent/memory_manager.py`
- **Pattern:** Provider ABC with lifecycle: initialize → system_prompt_block → prefetch → sync_turn → shutdown
- **Built-in:** BuiltinMemoryProvider always active (MEMORY.md/USER.md)
- **External:** One external provider at a time (Honcho, Hindsight, etc.)
- **Plugin path:** `plugins/memory/<name>/`
- **Config:** `memory.provider` in config.yaml

### API Server (v0.7.0)
- **Location:** `gateway/platforms/api_server.py` (2436 lines)
- **Open WebUI compatible:** OpenAI-compatible chat completions endpoint
- **Session continuity:** X-Hermes-Session-Id header for persistent sessions
- **Tool streaming:** Real-time tool progress events
- **Cron management:** Full CRUD via REST API

## What AgentJack Should Take

### P0 — Direct Pattern Adoption

1. **Profile isolation for tenants**
   - Hermes profile = AgentJack tenant
   - Each tenant gets: own config, memory, sessions, skills, workspace
   - Our current approach: single workspace, no tenant isolation
   - **Action:** Design AgentJack tenant directory structure similar to profiles

2. **Web dashboard architecture**
   - Hermes: React + FastAPI
   - AgentJack: SvelteKit + Python API server (already close)
   - **Gap:** AgentJack needs proper API layer for config, sessions, cron management
   - **Action:** Add REST API endpoints matching Hermes's `/api/*` pattern

3. **Pluggable memory**
   - Hermes: ABC with lifecycle hooks
   - AgentJack: hardcoded to OpenViking/Convex
   - **Action:** Define MemoryProvider ABC in AgentJack, make OpenViking one implementation

### P1 — Feature Parity

4. **Credential pool rotation** (v0.7.0)
   - Multiple API keys per provider, automatic rotation on 401
   - Critical for multi-tenant: each tenant has different quotas
   - **Action:** Design credential pool per tenant

5. **MCP server mode** (v0.6.0)
   - Expose AgentJack as MCP server for external clients
   - Accept MCP servers from integrations (v0.7.0)
   - **Action:** Add MCP adapter to AgentJack runtime

6. **Context engine plugin** (v0.9.0)
   - Swap context management strategy per deployment
   - Different tenants → different context strategies
   - **Action:** Define ContextEngine interface

### P2 — Security & Hardening

7. **Secret exfiltration blocking** (v0.7.0) — scan browser URLs and LLM responses
8. **Path traversal protection** (v0.9.0) — checkpoint manager, file operations
9. **Shell injection neutralization** (v0.9.0) — sandbox writes
10. **Webhook signature validation** (v0.9.0) — all platform webhooks

## AgentJack Advantages Over Hermes

1. **SvelteKit frontend** — lighter than React, SSR-capable, better for B2B
2. **Python CLI bridge** — `python -m agentjack dispatch-task` is simpler than Hermes's gateway
3. **Multi-app architecture** — AgentJack apps (Courtroom, Automation) are unique
4. **Already has section-based routing** — `/automations`, `/brain`, `/integrations`

## What NOT to Copy

- **16 messaging platforms** — AgentJack is B2B tool, not chat consumer
- **Termux/Android** — not our target
- **Camofox browser** — anti-detection is niche
- **Research/RL tools** — batch trajectories, Atropos are research-specific
- **Nous Tool Gateway** — their monetization, not ours

## Immediate Action Items

1. Study `hermes_cli/profiles.py` tenant isolation pattern → adapt for AgentJack
2. Study `hermes_cli/web_server.py` API pattern → add similar endpoints to AgentJack
3. Study `agent/memory_provider.py` ABC → define AgentJack equivalent
4. Review `web/src/` React components → identify UI patterns worth porting to Svelte
