# AgentJack + Pi-Style Single-Workflow Agents Insight — 2026-04-26

## Matej's Insight

AgentJack may fulfill the long-running vision of self-developing business tools by combining:

1. AgentJack as the contained orchestrator/control plane.
2. Proprietary Pi-style single-workflow agents as narrow, persistent specialists.
3. Downloaded/open-source projects as proven starting substrates.
4. AgentJack built-in apps as tools exposed to those agents.
5. Kraliki/open-kraliki automation patterns as the execution substrate and reusable workflow library.

The shape is: **AgentJack → Pi-style workflow agent → isolated project/workflow package → improved business tool**, with AgentJack apps providing tools, memory, status, steering, and customer-facing surfaces.

## Why This Matters

This connects several previously separate threads:

- AgentJack is the main product and revenue vehicle: develop agent core first, then apps on top.
- Automations is already a first-class AgentJack app/vertical skin.
- open-kraliki is already the proven repo-local automation substrate behind Automations.
- The product should keep stable app boundaries while allowing experimental workbench internals behind narrow contracts.
- Matej prefers Pi-style specialists for repetitive long-horizon narrow tasks where agents can self-improve over time.

## Product Thesis

AgentJack should not try to be one giant general-purpose auto-dev monolith.

Instead, AgentJack should become a **business-tool foundry**:

- one contained customer/runtime box,
- many narrow workflow agents,
- each agent owns one durable business workflow,
- each workflow can start from an existing open-source project/template,
- AgentJack provides the app/tool surface around it: Chat, Automations, Brain, Council, Human Tasks, integrations, monitoring, billing, deployment.

## Premises

1. **Narrow agents beat broad agents for durable business tooling.** A persistent specialist that only improves one workflow can compound skill and memory faster than a general autodev loop.
2. **AgentJack's built-in apps are the tool belt.** The apps are not just UI modules; they become callable capabilities for workflow agents.
3. **Open-source projects reduce blank-page risk.** Instead of asking an agent to invent a full business tool from scratch, AgentJack can ingest a proven repo/template and improve/adapt it.
4. **open-kraliki remains substrate, not product UI.** Product-facing control stays in AgentJack Automations; repo-local packages remain implementation units behind contracts.
5. **The first wedge must be one business workflow, not the full foundry.** The fastest validation is one contained workflow agent that upgrades a real business tool and exposes it through AgentJack.

## Recommended First Wedge

Use a real, narrow workflow with revenue/usefulness pressure:

**"Landing-page + lead-capture optimizer agent"**

Inputs:
- existing website repo or open-source landing template,
- target customer profile,
- offer/copy constraints,
- analytics/conversion observations.

Agent tools:
- Git repo access,
- AgentJack Brain/memory,
- Automations task runner,
- CouncilNow-style critique/review,
- deployment/check gates,
- human approval for external/public changes.

Outputs:
- improved landing page branch/PR,
- conversion hypothesis,
- deployed preview,
- QA evidence,
- follow-up task list.

Why this wedge:
- business value is obvious,
- scoped enough for a single-workflow specialist,
- reuses AgentJack apps immediately,
- produces visible demos,
- maps directly to CouncilNow/AgentJack revenue work.

## Architecture Sketch

```text
AgentJack Chat / Automations
        |
        v
Workflow Agent Registry
        |
        v
Single Workflow Agent
        |
        +--> Project Adapter: repo/template/open-source project
        +--> Tool Adapter: AgentJack apps as callable tools
        +--> Memory Adapter: workflow-specific learning history
        +--> Safety Adapter: approval gates and publish policy
        +--> Evidence Adapter: tests, previews, screenshots, diffs
```

## Design Rule

Do not expose raw autodev/workbench internals to customers.

Expose:
- Workflow name
- Status
- Inputs
- Outputs
- Evidence
- Approval gates
- History
- Learned improvements

Hide:
- raw cron loops
- brittle scripts
- model/provider guts
- internal logs unless needed for debugging

## Open Question

Should the first workflow-agent prototype be framed as:

1. **Internal dogfood** — build our own CouncilNow/AgentJack growth tooling first.
2. **Customer-facing demo** — a polished public "business tool builder" demo.
3. **Partner-service wedge** — sell as a done-with-you automation/productization sprint.

Recommendation: start with **internal dogfood**, then package the best workflow as a partner-service wedge.
