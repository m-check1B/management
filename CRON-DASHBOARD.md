# Cron Dashboard

_Last updated: 2026-04-18 23:21_

## Strategic Crons (NEW)

| Job | Schedule | Status | Purpose |
|-----|----------|--------|---------|
| **axis-setup-step** | Daily 23:00 | ✅ New | Advance setup until fully operational |
| **councilnow-business-ops** | Weekdays 07:00 | ✅ New | Business health check |
| **agentjack-dev-sprint** | Daily 01:00 | ✅ New | Nightly AgentJack dev push |
| **gstack-gbrain-update-check** | Mondays 22:00 | ✅ New | Pull latest from both repos |
| **workspace-github-push** | Every 4h | ✅ New | Auto-push workspace to GitHub |

## Operational Crons

| Job | Schedule | Status | Errors | Purpose |
|-----|----------|--------|--------|---------|
| **Kraliki Patrol** | Every 6h | ✅ OK | 0 | Infrastructure patrol |
| **qa-hub-admin** | Every 30min | ✅ OK | 0 | Hub route health |
| **linear-triage** | Hourly :15 | ✅ OK | 0 | Issue triage |
| **Daily Log Review** | Daily 02:00 | ✅ OK | 0 | Log error review |
| **Doc Drift Checker** | Daily 03:00 | ✅ OK | 0 | Doc accuracy |
| **overnight-maintenance** | Daily 03:00 | ✅ Fixed | Was 8 | Memory consolidation, cleanup |
| **daily-backup** | Daily 04:00 | ✅ Fixed | Was 8 | Multi-layer backup |
| **Apple Notes Export** | Daily 04:00 | ✅ Fixed | Was 7 | Incremental notes export |
| **OpenClaw Discovery** | Daily 09:00 | ✅ Fixed | Was 8 | X/Twitter + web research |
| **OpenClaw Doctor** | Daily 08:00 | ✅ Fixed | Was 8 | Health check |
| **Mac Security Check** | Daily 08:00 | ✅ Fixed | Was 7 | Security audit |
| **GitHub Changes** | Every 6h | ✅ Fixed | Was 28 | Commit monitoring |
| **automation-ops-review** | Every 30min | ✅ Fixed | Was 34 | Autodev dashboard check |
| **vertical-healthcheck** | Mondays 06:00 | ✅ Fixed | Was 4 | Vertical health |

## Disabled / One-shot (Completed)

| Job | Status | Notes |
|-----|--------|-------|
| **Weekend Strategy Deep-Dive** | ⛔ Disabled | Ran 2026-03-21 |
| **Linear Fixer Sprint** | ⛔ Disabled | Was erroring |
