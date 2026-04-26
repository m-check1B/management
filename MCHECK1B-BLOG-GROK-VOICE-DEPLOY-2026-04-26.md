# m-check1B blog Grok Voice rollout — 2026-04-26

## Summary

Added Grok Voice API readings to the public m-check1B blog so articles can be consumed as podcast-style audio in English, Spanish, and Czech.

## Implementation

Repo: `/Users/matejhavlin/github/blog`

Added:

- `scripts/generate-grok-voice.mjs` — batch generator for xAI `/v1/tts` MP3 readings, with chunking for >15k character essays, ffmpeg concat, retry handling for transient 429/5xx failures, sidecar metadata, and podcast RSS generation.
- `src/lib/content/audio.ts` + `src/lib/content/audio.json` — typed audio manifest consumed by Svelte pages.
- Article page audio player with:
  - native `<audio controls>` player
  - duration and Grok voice metadata
  - MP3 download link
  - per-language Podcast RSS link
  - PostHog events for play/download/RSS clicks.
- Homepage Podcast RSS link and featured article `Listen` CTA.
- Static podcast feeds:
  - `/podcast-en.xml`
  - `/podcast-es.xml`
  - `/podcast-cs.xml`
- Static audio assets under `/audio/blog/<slug>/<lang>.mp3`.

## Generated audio

- Provider: xAI Grok Voice API
- Voice: `rex`
- Codec: MP3, mono, 24 kHz, 64 kbps
- Languages generated: English (`en`), Spanish (`es`), Czech (`cs`)
- Tracks generated: 27 total — 9 posts × 3 languages
- Largest file: `aq-what-future-holds/es.mp3` at ~13.14 MiB / ~28.7 minutes
- Total source audio size: ~156 MiB

Note: initial 128 kbps output was too close to static-hosting file-size risk for long essays, so the generator was switched to 64 kbps before final generation.

## Build

Passed locally:

```bash
npm run check
npm run build
npm run test:static-html
```

Results:

- `svelte-check` → 0 errors, 0 warnings
- SvelteKit static build → passed
- Static HTML validity → passed for 34 pages

## Deploy

Deployed via Cloudflare Pages using the existing Pages deployment path:

```bash
npx wrangler pages deploy build --project-name m-check1b --branch main
```

Deployment URL:

- `https://c1dccaab.m-check1b.pages.dev`

Production URL:

- `https://m-check1b.pages.dev/`

Wrangler result:

- Uploaded 111 files, 38 already uploaded
- Deployment complete

## Public verification

Verified production with browser-style User-Agent:

- `https://m-check1b.pages.dev/en/` → 200, contains Podcast RSS link
- `https://m-check1b.pages.dev/en/blog/how-not-to-be-pre-sorted-by-the-future/` → 200, contains audio player and audio URL
- `https://m-check1b.pages.dev/es/blog/how-not-to-be-pre-sorted-by-the-future/` → 200, contains audio player and audio URL
- `https://m-check1b.pages.dev/cs/blog/how-not-to-be-pre-sorted-by-the-future/` → 200, contains audio player and audio URL
- `https://m-check1b.pages.dev/podcast-en.xml` → 200, contains RSS enclosures with `audio/mpeg`
- `https://m-check1b.pages.dev/podcast-es.xml` → 200, contains RSS enclosures with `audio/mpeg`
- `https://m-check1b.pages.dev/podcast-cs.xml` → 200, contains RSS enclosures with `audio/mpeg`

Audio asset checks:

- `https://m-check1b.pages.dev/audio/blog/how-not-to-be-pre-sorted-by-the-future/en.mp3` → 200, `content-type: audio/mpeg`, `content-length: 11653293`, file reports MP3 64 kbps / 24 kHz / mono
- `https://m-check1b.pages.dev/audio/blog/aq-what-future-holds/es.mp3` → 200, `content-type: audio/mpeg`, `content-length: 13780461`, file reports MP3 64 kbps / 24 kHz / mono

Browser snapshot verified the live article exposes:

- `Voice reading` region
- `24 min 17 sec`
- `Grok voice · REX`
- `Podcast RSS`
- `Download MP3`

## Source control note

The blog repo has pre-existing unrelated dirty/untracked import work and no configured `origin` remote. This rollout is deployed live from the local working tree. Avoid broad `git add .` in this repo until the existing import/migration state is intentionally cleaned or committed.
