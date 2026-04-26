# m-check1B personal site deployment — 2026-04-26

## Summary

Created and deployed the personal AI consultant site for `m-check1B` from `/Users/matejhavlin/github/blog`.

## Build

- `npm run check` → 0 errors, 0 warnings
- `npm run build` → passed with SvelteKit adapter-static output to `build/`

## Deploy

- Cloudflare Pages project created: `m-check1b`
- Deployed `build/` with Wrangler using the existing Cloudflare Pages token.
- Production Pages URL: `https://m-check1b.pages.dev/`
- Deployment URL observed: `https://b6da7f54.m-check1b.pages.dev`

## Verify

Public URL checks passed:

- `https://m-check1b.pages.dev/en/` → 200, title `m-check1B — AI Systems Consultant`
- `https://m-check1b.pages.dev/cs/` → 200, title `m-check1B — AI Systems Consultant`
- `https://m-check1b.pages.dev/es/` → 200, title `m-check1B — AI Systems Consultant`

Cloudflare Pages custom domains were added and are pending DNS:

- `m-check1b.com`
- `www.m-check1b.com`

## DNS blocker

Domain is registered at Namecheap and currently uses Namecheap nameservers:

- `DNS1.REGISTRAR-SERVERS.COM`
- `DNS2.REGISTRAR-SERVERS.COM`

Namecheap API update is blocked because the account API whitelist contains `193.179.119.136`, while the current request IP is `193.179.119.168`.

Required Namecheap DNS records:

| Type | Host | Value | TTL |
| --- | --- | --- | --- |
| ALIAS | `@` | `m-check1b.pages.dev` | Automatic |
| CNAME | `www` | `m-check1b.pages.dev` | Automatic |

After DNS is set, verify:

```bash
dig +short m-check1b.com
dig +short www.m-check1b.com CNAME
curl -I https://m-check1b.com/
curl -I https://www.m-check1b.com/
```

## Blog commits

- `bec413c` — Create personal AI blog
- `e801435` — Position site as AI consultant brand
- `e807393` — Document m-check1B Pages deployment
## Essay loading fix

Matej reported essays were not loading after first deploy. Root cause was invalid generated HTML: `<html lang={lang}></html>` was placed inside Svelte `<svelte:head>`, producing nested `<html>` tags in `<head>`. Browsers reparsed the page and hydration emptied the body.

Fix deployed from blog commit `bf50154` to Cloudflare Pages deployment `https://2249c45c.m-check1b.pages.dev` and production `https://m-check1b.pages.dev`. Gates: `npm run check`, `npm run build`, `npm run test:static-html`, public curl for homepage/articles, and browser article load with no console errors.

