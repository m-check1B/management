# Decisions Log

_Newest on top. Newer always overrides older._

## 2026-04-26 08:39 — AgentJack runtime ownership + side-by-side baseline

- **AgentJack runtime ownership locked to dedicated gateway:** `agentjack-gateway.service` is canonical; `hermes-gateway-axis.service` stays disabled for AgentJack Telegram lane.
- **Telegram 409 conflict classified as process-ownership bug, not token mismatch.**
- **OpenClaw vs AgentJack side-by-side audit baseline created** and stored in management (`OPENCLAW-vs-AGENTJACK-AUDIT-2026-04-26.md`) and mirrored to Linear (`KRA-3680`).
- **Near-term priority order from audit:**
  1. OpenClaw security criticals
  2. AgentJack private webchat readyz recovery
  3. AgentJack integrations extraction maturity


## 2026-04-19 07:32 — 20 Questions Answered

### Product & Revenue
- **CouncilNow pricing:** TBD — needs to be figured out
- **Stripe:** Exists, not tested end-to-end with real payment
- **CouncilNow target:** Business intelligence first → broaden later. Founders, vibe coders, people with complex decisions, automated decision making at scale. **Trading** as vertical.
- **councilnow.com:** DNS pointing to dev-2026 already
- **Revenue target:** "We will see" — no fixed number

### AgentJack
- **Elevator pitch:** "Not a toy but business weapon"
- **Target customer:** B2B — single to small to medium and up pattern
- **vs OpenClaw:** Answered in chat (need to capture)
- **Current state:** Backend significantly more advanced than UI allows. Agent kind of workable. UI not really wired.
- **First apps:** (1) Courtroom (2) Automation — almost completely done if based on open-kraliki

### Infrastructure
- **Zitadel:** Works with Google, not tested without Google
- **dev-2026:** Full root/admin access
- **DNS/Cloudflare:** Needs reconfiguration. Roadmap exists in infra — find it.
- **Hetzner:** Fine, no cost concerns
- **Mail server:** Should be redone. Sites on Cloudflare websites, rest on dev-2026. VM shut down or hot backup for databases.

### Operations
- **Money out:** Axis does NOT make payments. Ever. Matej decides.
- **Comms:** Axis does all marketing when strategy is ready (not yet). Telegram to Matej for updates.
- **Availability:** Async via Telegram, onsite daily in webchat.
- **Autodev:** Axis can use old autodev as a tool, ONLY on AgentJack (not CouncilNow). Pre-production only, never on production.
- **Weekend:** Possibly working today

### UI/Harness Decision (APPROVED 07:20)
- Keep AgentJack SvelteKit webchat
- Port GoClaw B2B admin patterns (tenants, teams, API keys, usage)
- Study GoClaw architecture, reimplement in Python
- GoClaw CC BY-NC — patterns only, no code

## 2026-04-18 — Axis Reset & Strategy Lock

- **Axis = CEO, Software Developer & Product Director. Matej = Founder.** Axis owns build, architecture, sprints. Matej owns vision, funding, strategy.
- **Top-level plan:** (1) Setup Axis fully → (2) Run CouncilNow as real business → (3) Develop AgentJack as advanced B2B → (4) AgentJack must surpass Garry's stack
- **Product ≠ tooling.** OpenClaw/GStack/GBrain = workbench (internal). AgentJack = product (external).
- **Coding stack locked:** GLM-5.1 (Axis direct), GPT-5.4 xhigh (Codex coding), GPT-5.4 chigh (Codex non-coding), Kimi CLI (web design), Grok 4.3 (current info)
- **Push to GitHub regularly.** Git is undo. Walk back if needed.
- **GStack = primary cookbook.** GBrain = knowledge infra. Both fast-evolving — stay current.
- **Kraliki monorepo = heritage, stopped.** Mine it, don't build it.
- **open-kraliki = automation templates.** Substrate for AgentJack.
- **Scattered memories need consolidation.** Newer wins over older on conflicts.
- **Management hub created** at `/github/management/` — single source of truth for Matej.

## Earlier Decisions (from previous Axis lifetimes)

_See MEMORY.md and obsidian-memory/ for historical context._
