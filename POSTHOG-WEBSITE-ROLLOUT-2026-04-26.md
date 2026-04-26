# PostHog website rollout — 2026-04-26

## Summary

Updated PostHog analytics across the public website surfaces and the new personal blog with consent-gated initialization, safer privacy handling, deployment-time CouncilNow key injection, and live verification where a production target exists.

## Scope

- `kraliki.com`
- `verduona.com`
- `perfectmission.co.uk`
- `councilnow.com` marketing page served through the Advisory Council app deploy
- `m-check1b.pages.dev` personal blog
- `agentjack.kraliki.com` static marketing page source

## Build

- `kraliki.com`: `npm run check` passed with 0 errors and 1 pre-existing warning in `ChatWidget.svelte`.
- `verduona.com`: `npm run check` passed with 0 errors/warnings.
- `perfectmission.co.uk`: `npm run check` passed with 0 errors/warnings.
- Blog: `npm run check`, `npm run build`, and `npm run test:static-html` passed; static HTML validity covered 34 pages after privacy route addition.
- AgentJack static: `npm run build` passed.
- Static HTML: `htmlhint` passed for `agentjack.kraliki.com/index.html` and `councilnow.com/index.html`.
- Websites deploy dry-run built `kraliki`, `verduona`, and `perfectmission` successfully.

## Deploy

- `kraliki.com`: deployed via `~/.openclaw/workspace/skills/websites-deploy/deploy.sh`; `websites-kraliki-1` restarted.
- `verduona.com`: deployed via `~/.openclaw/workspace/skills/websites-deploy/deploy.sh`; `websites-verduona-1` restarted.
- `perfectmission.co.uk`: initial rsync failed because the remote web root was owned by UID `501:staff`; fixed ownership/permissions on `/home/adminmatej/websites/perfectmission`, redeployed via the canonical deploy script, and restarted `websites-perfectmission-1`.
- `councilnow.com`: deployed Advisory Council app with tag `posthog-marketing-202604261600`; deploy script now injects the live PostHog key from `/Users/matejhavlin/github/secrets/posthog_api_key.txt` or `POSTHOG_API_KEY` and restores the tracked placeholder file after deploy.
- Blog: deployed to Cloudflare Pages using the archived Pages token; deployment URL `https://cec06623.m-check1b.pages.dev`, production URL `https://m-check1b.pages.dev`.
- AgentJack static page was updated and built locally, but not publicly deployed because `agentjack.kraliki.com` has no resolving DNS/canonical deployment target yet.

## Verify

- `https://kraliki.com/` returned HTTP 200 after server file permissions were corrected; Svelte assets contain PostHog code in 2 JS assets.
- `https://verduona.com/` returned HTTP 200 after server file permissions were corrected; Svelte assets contain PostHog code in 1 JS asset.
- `https://perfectmission.co.uk/` returned HTTP 200 after ownership/permission fix; Svelte assets contain PostHog code in 4 JS assets.
- `pnpm verify:live https://councilnow.com/` passed: live marketing HTML has PostHog config, EU host, expected CTA locations/labels, and no unresolved placeholder key.
- `https://councilnow.com/health` returned 200 with `status=ok`; `https://councilnow.com/app` returned 200.
- `https://m-check1b.pages.dev/en/` and `https://cec06623.m-check1b.pages.dev/en/` returned 200 with title `m-check1B — AI Systems Consultant`; both serve PostHog/cookie-consent code in 2 JS assets.

## Commits

- `websites`: `bbf02cf Update marketing PostHog consent instrumentation` pushed to `main`.
- `perfectmission-website`: `477ac35 Harden PerfectMission PostHog consent handling` pushed to `main`.
- `advisory-council-app`: `d24c10b Inject CouncilNow marketing PostHog key during deploy` pushed to `master`.
- `blog`: `db23aaf Add consent-gated PostHog to personal blog` committed locally; no `origin` remote is configured, but the Cloudflare Pages deployment succeeded.

## Blockers / notes

- `agentjack.kraliki.com` remains blocked on DNS/deployment target. Current DNS lookup does not resolve.
- Blog Git push is blocked because `/Users/matejhavlin/github/blog` has no `origin` remote configured.
- Pre-existing dirty blog content/import work and pre-existing PerfectMission local changes were intentionally left untouched.
- The websites deploy helper was patched to set web-readable permissions after rsync so Apache/httpd does not return 403 on freshly deployed private-mode build outputs.
