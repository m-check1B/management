# Website Lineup + CouncilNow Cross-links — 2026-04-26

## Status
Done and deployed to the current production host (`91.99.176.2`).

## Changes
- `kraliki.com`
  - Updated root redirect SEO metadata from old “AI Swarm Platform” wording to current agent-systems positioning.
  - Strengthened CouncilNow cross-reference in the commercial split copy: live at `councilnow.com` and positioned as the decision engine above AgentJack.
- `verduona.com`
  - Updated root redirect SEO metadata from old “AI Delivery Platform” wording to current products/training positioning.
  - Added direct “Open CouncilNow” CTA to the hero.
  - Updated hero note to state CouncilNow is the live decision product in the portfolio.
- Deployment docs
  - Updated website deploy instructions to use rsync chmod and post-deploy `chmod -R a+rX`; production Apache returned 403 until permissions were corrected.

## Build gates
- `cd kraliki.com && npm run check && npm run build`
  - Result: build passed.
  - Existing warning remains in `ChatWidget.svelte`: `voiceController` non-reactive update. No new errors.
- `cd verduona.com && npm run check && npm run build`
  - Result: 0 errors, 0 warnings; build passed.

## Deploy
- Synced `kraliki.com/build/` to `/home/adminmatej/websites/kraliki/`.
- Synced `verduona.com/build/` to `/home/adminmatej/websites/verduona/`.
- Fixed permissions and restarted:
  - `websites-kraliki-1`
  - `websites-verduona-1`

## Public verification
- `https://kraliki.com/en/`
  - HTTP `200`
  - Contains `CouncilNow is already proving the decision layer`
  - Contains `live at councilnow.com`
- `https://verduona.com/en/`
  - HTTP `200`
  - Contains `Open CouncilNow`
  - Contains `CouncilNow is the live decision product`
  - Contains `councilnow.com/app`

## Blockers
None for this scoped task.
