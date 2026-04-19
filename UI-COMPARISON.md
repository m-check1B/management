# UI Comparison: OpenClaw vs GoClaw vs AgentJack

_Created: 2026-04-19 07:20 by Axis_

## The Three UIs

### 1. OpenClaw Control UI (what we use now)
- **Stack:** React (embedded in OpenClaw gateway)
- **What it does:** Chat, settings, health, basic agent management
- **Quality:** Works, minimal, functional for single-user
- **B2B ready:** ❌ No multi-tenant, no admin panel, no billing UI
- **Verdict:** Our internal workbench. Not a product UI.

### 2. GoClaw Web Dashboard (React + TypeScript)
- **Stack:** React 19, Vite, Tailwind, TanStack Query, @xyflow/react
- **Pages (32!):**
  - **Core:** chat, agents, sessions, overview, providers, config
  - **Admin:** tenants-admin, teams, users, api-keys, storage
  - **Dev tools:** cron, skills, mcp, knowledge-graph, memory, logs, traces
  - **Ops:** channels, nodes, approvals, pending-messages, activity, events
  - **Infra:** tts, builtin-tools, cli-credentials, import-export, packages, usage
- **Also has:** Desktop app (Wails v2, macOS + Windows), i18n (32 languages)
- **Quality:** Production-grade, complete admin dashboard
- **B2B ready:** ✅ Multi-tenant admin, team management, API keys, usage tracking
- **Verdict:** Best UI of the three. Complete B2B admin panel. React-based.

### 3. AgentJack (two SvelteKit apps)
**ag-webui** (SvelteKit 5):
- **Stack:** SvelteKit, Svelte 5, Tailwind 4
- **Pages:** overview, chat, health, admin, readyz
- **Quality:** Early stage, basic
- **B2B ready:** ❌ Too early

**webchat** (SvelteKit 5, the main one):
- **Stack:** SvelteKit, Svelte 5, Tailwind 4
- **Pages (20+):** chat, automations, brain, channels, cron, dashboard, events, human, logs, mcp, memory, obsidian, pending, plugins, providers, sessions, skills, swarm, traces, voice, workspaces, billing, auth
- **Quality:** Good foundation, actively refactored (chat page went from 6,468 → 2,810 lines)
- **B2B ready:** Partial — has billing and auth routes, but no multi-tenant admin
- **Unique:** Brain page, automations designer, voice, obsidian integration

## Side-by-Side

| Feature | OpenClaw | GoClaw | AgentJack webchat |
|---------|----------|--------|-------------------|
| Framework | React | React | **SvelteKit** |
| Multi-tenant admin | ❌ | ✅ | ❌ |
| Team management | ❌ | ✅ | ❌ |
| Chat UI | ✅ | ✅ | ✅ (best refactored) |
| Agent management | Basic | Full | Full |
| Cron/scheduling | Basic | ✅ | ✅ |
| MCP management | ❌ | ✅ | ✅ |
| Knowledge graph | ❌ | ✅ | Brain (partial) |
| Skills management | ✅ | ✅ | ✅ |
| Billing/payments | ❌ | ❌ | ✅ (Stripe routes) |
| Auth/SSO | ❌ | Login page | ✅ (Zitadel) |
| Memory browser | ❌ | ✅ | ✅ |
| Traces/observability | ❌ | ✅ | ✅ |
| Voice UI | TTS only | TTS (4 providers) | ✅ Full voice |
| Automation designer | ❌ | ❌ | ✅ Unique |
| Brain/council UI | ❌ | ❌ | ✅ Unique |
| Obsidian integration | ❌ | ❌ | ✅ Unique |
| Desktop app | ✅ (macOS/iOS/Android) | ✅ (macOS/Windows) | ❌ |
| i18n | ❌ | ✅ (32 languages) | ❌ |
| Mobile responsive | Basic | ✅ | Partial |
| Total pages/routes | ~8 | **32** | **20+** |

## Honest Assessment

**GoClaw's UI is the most complete B2B dashboard.** 32 pages covering everything a platform operator needs — tenant management, team boards, API key lifecycle, usage tracking, knowledge graph browser. It's production-grade React.

**AgentJack's webchat has unique product value** that GoClaw doesn't: Brain page, automation designer, voice interface, obsidian integration, billing/auth already wired. The architecture is cleaner (agent-centric refactor done).

**OpenClaw's Control UI is the weakest** — fine for personal use, not a product.

## Recommendation: What to Do with the UI

### Don't Switch Frameworks
We're invested in SvelteKit. Switching to React (GoClaw's stack) would be a complete rewrite for zero product gain. SvelteKit 5 + Tailwind 4 is modern and fast.

### Do This Instead

**Phase 1: Copy GoClaw's Pages, Keep Our Stack (2-3 weeks)**

Port the missing B2B pages from GoClaw into AgentJack's webchat as SvelteKit routes:

| GoClaw Page | AgentJack Equivalent | Priority |
|-------------|---------------------|----------|
| tenants-admin | **BUILD NEW** | P0 — multi-tenant admin |
| teams | **BUILD NEW** | P0 — team management |
| api-keys | **BUILD NEW** | P0 — customer API key lifecycle |
| knowledge-graph | Enhance Brain | P1 |
| storage | **BUILD NEW** | P1 |
| usage | **BUILD NEW** | P1 — usage/billing dashboard |
| activity | **BUILD NEW** | P2 |
| import-export | **BUILD NEW** | P2 |
| contacts | **BUILD NEW** | P2 |
| i18n | Add later | P3 |

**Phase 2: Keep What's Unique to AgentJack (already built)**
- ✅ Brain page (our unique product)
- ✅ Automation designer (our unique product)
- ✅ Voice interface
- ✅ Billing/Stripe routes
- ✅ Auth/Zitadel integration
- ✅ Council integration (coming via strategy)

**Phase 3: Desktop App (Month 3+)**
- GoClaw uses Wails (Go + React). We'd use Tauri (Rust + SvelteKit).
- Or skip desktop and ship web-first (most B2B products are web-only).
- Decision later — not blocking.

## What This Looks Like

```
AgentJack Webchat (SvelteKit 5)
├── EXISTING (keep)
│   ├── chat/          ← Agent-centric, refactored ✅
│   ├── brain/         ← Unique product ✅
│   ├── automations/   ← Unique product ✅
│   ├── voice/         ← Unique ✅
│   ├── billing/       ← Stripe ✅
│   ├── auth/          ← Zitadel ✅
│   ├── providers/     ← LLM config ✅
│   ├── sessions/      ← Session management ✅
│   ├── skills/        ← Skill system ✅
│   ├── mcp/           ← MCP management ✅
│   └── memory/        ← Memory browser ✅
│
├── NEW (port patterns from GoClaw)
│   ├── tenants/       ← Multi-tenant admin (P0)
│   ├── teams/         ← Team management (P0)
│   ├── api-keys/      ← Customer API keys (P0)
│   ├── usage/         ← Usage dashboard (P1)
│   ├── storage/       ← File/data management (P1)
│   ├── knowledge/     ← Enhanced brain + graph (P1)
│   └── settings/      ← Platform settings (P1)
│
└── LATER
    ├── i18n           ← Multi-language
    └── desktop/       ← Tauri app (if needed)
```

## TL;DR

- **Don't switch to GoClaw's React UI.** Our SvelteKit is better long-term and already has unique product pages.
- **Don't stick with OpenClaw's UI.** It's not a product.
- **Do study GoClaw's 32 pages** and port the B2B admin patterns (tenants, teams, API keys, usage) into our SvelteKit webchat.
- **Keep building on AgentJack's webchat.** It has the most product value (Brain, Automations, Voice, Billing, Auth) that neither competitor has.
- **The UI is a strength, not a weakness.** We just need the B2B admin pages.
