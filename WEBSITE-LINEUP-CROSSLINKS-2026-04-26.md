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

---

## Visual Restoration Follow-up — 2026-04-26

Matej rejected the flatter new rewrite direction for the marketing sites. Restored the richer previous visual language while keeping only surgical current naming and cross-link updates.

### Restoration changes
- `kraliki.com`
  - Restored developed visual components from the earlier richer version: animated/dark hero, manifesto, process circle, testimonials, and CTA section.
  - Kept current naming in the restored visual shell: AgentJack, CouncilNow, Kraliki stack, decision/execution/memory positioning.
  - Preserved cross-links:
    - `Launch AgentJack` → `https://agentjack.kraliki.com`
    - `CouncilNow` / `Open CouncilNow` → `https://councilnow.com` / `https://councilnow.com/app`
- `verduona.com`
  - Restored the richer older hero treatment.
  - Kept current portfolio naming: CouncilNow, AgentJack, Notes, Garage, Security, Courseroom.
  - Changed secondary hero CTA to `Open CouncilNow` → `https://councilnow.com/app`.

### Commit + deploy
- Websites commit: `602c3a6 Restore developed marketing visuals`
- Pushed to `origin/main`.
- Deployed via `rsync --delete --chmod=Du=rwx,Dgo=rx,Fu=rw,Fgo=r` to:
  - `/home/adminmatej/websites/kraliki/`
  - `/home/adminmatej/websites/verduona/`
- Ran `chmod -R a+rX` and restarted:
  - `websites-kraliki-1`
  - `websites-verduona-1`

### Gates
- `cd kraliki.com && npm run check && npm run build`
  - Passed with existing unrelated `ChatWidget.svelte` warning only.
- `cd verduona.com && npm run check && npm run build`
  - Passed with `0 errors / 0 warnings`.
- `git diff --check -- kraliki.com verduona.com`
  - Passed.

### Public verification
- `https://kraliki.com/en/?restore=602c3a6`
  - Contains `DECIDE. EXECUTE.`
  - Contains `LAUNCH AGENTJACK`
  - Contains `COUNCILNOW`
  - Browser snapshot confirmed rich restored hero + manifesto/process/testimonial layout.
- `https://verduona.com/en/?restore=602c3a6`
  - Contains `Products and training built on Kraliki systems`
  - Contains `EXPLORE PRODUCTS`
  - Contains `OPEN COUNCILNOW`
  - Browser snapshot confirmed restored hero treatment and current product portfolio below.

### Local visual evidence
- `/Users/matejhavlin/.openclaw/workspace/artifacts/kraliki-marketing-restore.png`
- `/Users/matejhavlin/.openclaw/workspace/artifacts/verduona-marketing-restore.png`
