# Infrastructure Map

_Last updated: 2026-04-18 23:21_

## Hardware

```
┌─────────────────────────────────────┐
│  Matej's MacBook Pro (ARM)          │
│  PRIMARY WORKSTATION                │
│  ├─ OpenClaw Gateway (Axis lives)   │
│  ├─ PM2 (16 processes)              │
│  ├─ Docker (gbrain-pg, litellm...)  │
│  ├─ Embedding server :1934          │
│  └─ Dev environment                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  dev-2026 (Hetzner)                 │
│  PRODUCTION SERVER                  │
│  ├─ VMs (various services)          │
│  ├─ PM2 (22 processes)              │
│  └─ Docker containers               │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│  Mail VM (Hetzner)                  │
│  EMAIL SERVER                       │
└─────────────────────────────────────┘
```

## DNS & Hosting

| Service | Provider | Purpose |
|---------|----------|---------|
| DNS/CDN | Cloudflare | All domains |
| Domains | Namecheap | Domain registration |
| Servers | Hetzner | dev-2026, mail VM |

## Key Services

| Service | URL/Port | Status |
|---------|----------|--------|
| Zitadel (SSO) | identity.verduona.dev | ✅ Active |
| Hub API | :8203 | ✅ Active |
| GBrain | localhost:5435 (Postgres) | ✅ Running |
| Embeddings | localhost:1934 | ✅ Running |
| OpenClaw Gateway | localhost (native) | ✅ Running |

## Domains

| Domain | Purpose | Where |
|--------|---------|-------|
| councilnow.com | CouncilNow product site | Cloudflare → ? |
| kraliki.com | Brand marketing | Cloudflare → ? |
| verduona.com | Brand marketing | Cloudflare → ? |
| identity.verduona.dev | Zitadel SSO | Cloudflare → dev-2026? |

## Backup Layers

1. GitHub — workspace + all repos
2. TimeMachine — Mac hourly
3. Hetzner — nightly server backup
4. Mac hard drive — local
5. Proton Drive — cloud (Axis folder)
6. Flash drive — manual hard backup
