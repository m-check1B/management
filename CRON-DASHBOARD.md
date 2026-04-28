# Cron Dashboard

_Last updated: 2026-04-28 17:12 CEST_

## Current policy

**Paused by founder instruction.** Matej said to stop for today and wait for a new start command.

- Total stored cron jobs: **39**
- Enabled cron jobs: **0**
- Previously enabled before pause: **21**
- Restore manifest: `/Users/matejhavlin/.openclaw/workspace/PAUSED-CRON-JOBS-2026-04-28.md`
- Pause marker: `/Users/matejhavlin/.openclaw/workspace/STOP_FOR_TODAY.md`
- Heartbeat behavior while paused: return `HEARTBEAT_OK`, do not resume old work.

## Why this changed

The cron set had accumulated too much old strategy/noise:

- duplicate AgentJack autodev cycles
- legacy `axis-*` reminders and selfchecks
- old one-shot follow-ups
- overly frequent checks (`linear-triage`, smoke tests, vector reindex)
- generic discovery/setup/infra cleanup jobs that can resurrect stale agendas
- announce-style daily jobs that can create visible chat noise

## Restart rule

Do **not** re-enable everything on resume.

When Matej explicitly says start/resume, rebuild the cron set from the restore manifest and keep only the jobs that serve the current live plan.

Recommended lean set:

| Job | Suggested schedule | Reason |
|-----|--------------------|--------|
| `workspace-github-push` | every 4–8h or daily | continuity/backup, low cognitive load |
| `daily-backup` | daily | self-preservation / data safety |
| `openclaw-doctor-daily` | daily or weekly | health baseline, only if quiet/no announce spam |
| `agentjack-autodev-cycle` | one canonical job only | product momentum; no duplicates |
| `gstack-gbrain-update-check` | weekly | keep cookbook + knowledge infra current |
| `Mac Security Check` | weekly | reasonable safety hygiene |
| `axis-revenue-nudge` | only while actively selling CouncilNow | avoid stale revenue reminder loops |

## Jobs paused on 2026-04-28

All 21 enabled jobs were disabled. Notable ones:

- `axis-infra-cleanup-nudge`
- `linear-triage`
- `agentjack-autodev-cycle`
- `axis-revenue-nudge`
- `github-changes-monitor`
- `Kraliki Patrol`
- `axis-selfcheck`
- `workspace-github-push`
- `OpenClaw Update Checker (KRA-3537)`
- `axis-setup-step`
- `x-watch-garry-matej`
- `agentjack-dev-sprint`
- `Daily Log Review (KRA-3534)`
- `Doc Drift Checker (KRA-3535)`
- `overnight-maintenance`
- `daily-backup`
- `Apple Notes Export (Daily)`
- `openclaw-doctor-daily`
- `Mac Security Check`
- `OpenClaw Discovery (X.com + Web)`
- `gstack-gbrain-update-check`

## Non-negotiable guardrail

Heartbeats and cron reminders are support infrastructure only. They must not outrank live founder direction or recreate an old operating plan.
