# Management Hub

Single source of truth for what's happening across all projects.
Axis maintains these files automatically. Matej reads them to stay current.

## Files

| File | What it shows | Updated |
|------|---------------|---------|
| `STATUS.md` | Current status of every project + infrastructure | Daily |
| `CRON-DASHBOARD.md` | All cron jobs, their health, last run status | Daily |
| `REVENUE-PIPELINE.md` | CouncilNow + AgentJack revenue tracking | Weekly |
| `SPRINT-CURRENT.md` | What Axis is working on RIGHT NOW | Real-time |
| `INFRA-MAP.md` | Hardware, servers, DNS, services — where everything lives | Weekly |
| `DECISIONS-LOG.md` | Key decisions with dates — newest on top | As needed |
| `OPENCLAW-vs-AGENTJACK-AUDIT-2026-04-26.md` | Side-by-side gap audit (runtime, security, integrations) | Snapshot (2026-04-26) |

## How to use

- **Quick check:** Read `STATUS.md` — 2 min, full picture
- **What's Axis doing?** Read `SPRINT-CURRENT.md`
- **Money?** Read `REVENUE-PIPELINE.md`
- **Infrastructure?** Read `INFRA-MAP.md`
- **Why was X decided?** Read `DECISIONS-LOG.md`

## Rules

- Axis writes, Matej reads (or asks)
- Newer info always wins over older
- Keep each file under 200 lines — scannable, not encyclopedic
- If a file is stale >3 days, the cron should have caught it
