# AgentJack External Verticals — Autonomous Lane

_Recorded: 2026-04-27 14:48 by TBA-One-PA_

## Founder direction

Matej clarified that while he travels, the easiest useful autonomous workstream is **the apps inside AgentJack — AgentJack's external verticals / sibling app boundaries**.

This matches the existing AgentJack product rule: AgentJack core is the stable orchestrator; product apps stay as separate app boundaries and are consumed through explicit contracts, not deep internal imports.

## Scope I can run without bothering Matej

Work inside `/Users/matejhavlin/github/agentjack` and adjacent proven substrates, especially:

1. **Automat / Automations**
   - Mature the external automation app boundary.
   - Reuse `open-kraliki` as the proven execution substrate.
   - Keep AgentJack runtime scheduling/dispatch separate from the external app UI.

2. **Brain / Council intelligence**
   - Improve AgentJack-facing Brain surfaces that consume CouncilNow / Advisory Council APIs.
   - Do not change CouncilNow billing or public sales promises without Matej.

3. **Catch Notes App**
   - Tighten notes/capture REST contracts, UX, tests, and agent-side helpers.

4. **Creative Studio / Avatars**
   - Polish wrapper contracts, registry status, persona/avatar workflows, and AG WebUI launch cards.

5. **Collective**
   - Advance feed/hubs/insights/search contract skeletons into usable demo flows.

6. **Comms Center**
   - Improve channel/dispatch/escalation/analytics contract surfaces, without sending real external messages unless explicitly approved.

7. **Courseroom / Security app**
   - Keep them independently buildable and app-boundary clean; focus on smokeable contracts before deep feature work.

## Safe autonomous actions

- Inventory app routes/contracts and mark gaps.
- Add or repair local smoke tests.
- Improve AG WebUI app launchers, status cards, and read-only/demo flows.
- Build contract adapters between AgentJack and app boundaries.
- Use local/dev-only seed data and mocks where needed.
- Run checks/builds/smokes and commit/push clean batches.
- Deploy to `agentjack-dev.verduona.dev` only after gates pass and the change is internal/dev-safe.

## Boundaries

- No billing/payment changes.
- No public outreach, posts, or external messages without Matej.
- No DNS/production cutovers unless directly required and safe.
- No deep coupling of product runtime to experimental workbench internals.
- Preserve app boundaries: explicit JSON/REST/CLI contracts over importing sibling internals.

## First execution order

1. **Map current verticals** from `APPS.md`, `products/*`, `plugins/*`, and AG WebUI app routes.
2. **Pick the highest-demo-value vertical**: Automations/Automat first unless a product gate says otherwise.
3. **Make one vertical visibly useful** in AG WebUI: status, launcher, example workflow, smoke test.
4. **Repeat for next vertical**: Brain, Catch Notes, Creative, Collective, Comms.
5. Keep proof packs: diff, test output, screenshot/browser proof where useful, and deploy evidence.

## Current note

AgentJack APPS.md already defines the portfolio boundary model. The work is not inventing architecture; it is making those app boundaries operational, testable, and demoable.
