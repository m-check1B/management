# Website Original Baseline Restore — 2026-04-26

## Reason

Matej corrected the direction: the old Kraliki/Verduona sites, especially Kraliki, were already visually optimized. The current-state additions should be surgical, not a broad rewrite/backfix.

## Change

Restored the original optimized marketing copy baseline from the pre-rewrite site and kept only narrow current-state edits:

- Kraliki hero/manifests/process copy restored to original tone:
  - `AI-Native Swarm Platform`
  - `YOUR AGENTS DO THE HEAVY LIFTING.`
  - `Everything is Different.`
- Kraliki surgical current-state CTAs retained:
  - `Open AgentJack` → `https://agentjack.kraliki.com`
  - `CouncilNow →` → `https://councilnow.com`
  - final CTA `Open CouncilNow` / `Explore AgentJack`
- Verduona hero tagline and vertical catalog copy restored to original baseline.
- Verduona surgical current-state CTA retained:
  - `Open CouncilNow` → `https://councilnow.com/app`
- Non-English pages no longer receive English overwrite strings for those restored hero/CTA areas; localized CTA labels were applied.

## Files

- `kraliki.com/messages/{cs,en,es,pl,sk}.json`
- `verduona.com/messages/{cs,en,es,pl,sk}.json`

## Build

- `cd kraliki.com && npm run check` → 0 errors, 1 pre-existing warning in `src/lib/components/chatbot/ChatWidget.svelte` about non-reactive `voiceController`.
- `cd kraliki.com && npm run build` → passed.
- `cd kraliki.com && node scripts/postbuild-spa.js` → passed.
- `cd verduona.com && npm run check` → 0 errors / 0 warnings.
- `cd verduona.com && npm run build` → passed.
- `git diff --check` → passed.

## Deploy

Deployed static builds with permission-safe rsync and restarted production containers:

```bash
rsync -az --delete --chmod=Du=rwx,Dgo=rx,Fu=rw,Fgo=r kraliki.com/build/ adminmatej@91.99.176.2:/home/adminmatej/websites/kraliki/
rsync -az --delete --chmod=Du=rwx,Dgo=rx,Fu=rw,Fgo=r verduona.com/build/ adminmatej@91.99.176.2:/home/adminmatej/websites/verduona/
ssh adminmatej@91.99.176.2 'chmod -R a+rX /home/adminmatej/websites/kraliki /home/adminmatej/websites/verduona && sudo docker restart websites-kraliki-1 websites-verduona-1'
```

Restart output:

```text
websites-kraliki-1
websites-verduona-1
```

## Verify

Public checks:

- `https://kraliki.com/en/` → HTTP 200
- `https://verduona.com/en/` → HTTP 200
- Live Kraliki page contains:
  - `YOUR AGENTS DO THE HEAVY`
  - `Everything is Different`
  - `Open AgentJack`
  - `Open CouncilNow`
- Live Verduona page contains:
  - `Your agents do the heavy lifting.`
  - `Contact Centrum`
  - `Open CouncilNow`

Browser proof captured after deployment from live public pages.

## Git

Websites repo:

- `203f2cd Restore original marketing copy baseline`
