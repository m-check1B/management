# Secrets Vault — Professional Setup Plan 2026-04-29

## Current State: Secret Sprawl

### Locations (50+ files, 89+ references)
| Location | Count | Type |
|---|---|---|
| `~/github/secrets/` | 50+ files | API keys, env files, credentials |
| VM `/etc/kraliki/*/env` | 5 env files | Zitadel, Hub, AgentJack, Runner |
| Docker compose files | 89 references | Service configs |
| OpenClaw config | Multiple | Bot tokens, API keys |
| `.env` files in repos | Scattered | App-specific secrets |

### Problems
- No central inventory of what exists
- No rotation policy
- No audit trail
- Secrets in plain text on disk
- No separation between environments
- VM env files manually managed

## Solution: Infisical (Self-Hosted)

### Already Running
- **URL:** https://vault.kraliki.com (just enabled!)
- **Instance:** mail-tba, Infisical v0.43.47 (platform v0.158.18)
- **Org:** Kraliki (49a0d354-54a7-484e-a64c-4d509fb5618f)
- **Project:** Kraliki Production (eb65971c-34d1-498d-ab8d-cd51687bdfa6)
- **Auth:** Machine identity token stored in `~/.kraliki-pa/infisical.json`
- **DNS:** vault.kraliki.com → 91.99.176.2 (mail-tba)
- **TLS:** Let's Encrypt (auto-renewed by Traefik)
- **CLI:** v0.43.60 installed on Mac

### Architecture
```
vault.kraliki.com (HTTPS)
    ↓
mail-tba Traefik (TLS termination)
    ↓
Infisical container (port 8080)
    ↓
PostgreSQL + Redis (containerized)
```

## Migration Plan

### Phase 1: Inventory & Categorize (today)
- [ ] List all secrets from ~/github/secrets/
- [ ] Categorize: API keys, service credentials, tokens, certificates
- [ ] Map which services consume which secrets
- [ ] Identify secrets that can be deprecated

### Phase 2: Import to Infisical (today/tomorrow)
- [ ] Create environments: prod, staging, dev
- [ ] Create secret groups: auth, billing, infra, ai-services, comms
- [ ] Import all active secrets
- [ ] Tag by service/owner

### Phase 3: VM Integration (this week)
- [ ] Install Infisical CLI on each production VM
- [ ] Create machine identities for each VM
- [ ] Replace env files with `infisical run` wrappers
- [ ] Update systemd services to use Infisical-sourced secrets

### Phase 4: CI/CD Integration (next week)
- [ ] GitHub Actions: use Infisical CLI for secret injection
- [ ] Docker Compose: replace hardcoded secrets with env injection
- [ ] Deploy scripts: fetch from Infisical before build

### Phase 5: Rotation & Audit (ongoing)
- [ ] Set rotation reminders for all API keys
- [ ] Enable audit logging
- [ ] Quarterly secret review

## Environment Structure
```
Kraliki Production
├── prod/
│   ├── auth/          (Zitadel, Hub, JWT signing keys)
│   ├── billing/       (Stripe keys, price IDs)
│   ├── infra/         (SSH keys, CF tokens, server creds)
│   ├── ai-services/   (OpenAI, Grok, Kimi, OpenRouter keys)
│   ├── comms/         (Telegram, Resend, email creds)
│   └── apps/          (CouncilNow, AgentJack specific)
├── staging/
│   └── (mirror of prod with staging values)
└── dev/
    └── (local dev overrides)
```

## Secret Groups

### auth (critical)
- Zitadel master key
- Hub JWT signing keys
- OAuth client secrets
- API gateway tokens

### billing (critical)
- Stripe live keys (secret, publishable, webhook)
- Stripe price IDs

### infra (critical)
- Cloudflare API tokens
- SSH keys
- Hetzner API tokens
- Server admin credentials

### ai-services (sensitive)
- OpenAI API key
- Grok/XAI API key
- OpenRouter API key
- Kimi API key
- Moonshot API key

### comms (sensitive)
- Telegram bot tokens
- Resend API key
- Email credentials

### apps (sensitive)
- CouncilNow-specific secrets
- AgentJack-specific secrets
- Hub configuration values

## Access Control

### Machine Identities (per-VM)
| VM | Identity | Access |
|---|---|---|
| prod-control-plane | pcp-reader | prod/auth, prod/billing, prod/infra |
| agentjack-control | ajc-reader | prod/ai-services, prod/apps, prod/comms |
| agentjack-runner-pool | arp-reader | prod/ai-services, prod/apps |
| dev-2026 (host) | host-reader | all prod/* |

### Human Access
| User | Role | Access |
|---|---|---|
| Matej | Admin | all environments, all secrets |
| TBA-One-PA | Reader | all prod/* (via machine identity) |

## Immediate Next Steps

1. ✅ vault.kraliki.com live (HTTPS + Traefik)
2. ✅ CLI authenticated
3. ✅ Project exists (Kraliki Production)
4. ✅ Inventory all current secrets (50+ files cataloged)
5. 🔲 Import to Infisical (BLOCKED: E2EE requires workspace key access)
6. 🔲 Set up VM machine identities
7. 🔲 Replace env file references

## E2EE Blocker (2026-04-29 15:25)

### Issue
Infisical v0.158.18 enforces end-to-end encryption for all secret operations. The API requires `secretKeyCiphertext`, `secretKeyIV`, `secretKeyTag`, `secretValueCiphertext`, `secretValueIV`, `secretValueTag` for every secret creation.

### Root Cause
The machine identity (bc1aec43) is NOT a member of the Kraliki Production workspace (members: 0). Without workspace membership, the identity can't access the workspace encryption key, and can't create or read secrets.

### Solution (requires Matej in web UI)
1. Go to https://vault.kraliki.com
2. Log in with axis@kraliki.com / hJpWCQtf9KxgOwW6IGvaSshn
3. Go to Project Settings → Members → Add Identity
4. Add the machine identity (bc1aec43-4124-4258-8aed-c26625f11ac3) as a member
5. The system will encrypt the workspace key with the identity's public key
6. After that, the CLI and API will work for secret CRUD

### Alternative (CLI on mail-tba)
The CLI on mail-tba is v0.38.0 (too old for E2EE). After adding identity to workspace:
- Install latest Go-based CLI on mail-tba
- Use `infisical login --method=universal-auth` with client credentials
- Then `infisical secrets set` will work

### SSH Tunnel Available
`ssh -L 8320:127.0.0.1:8320 root@91.99.176.2` → access web UI at http://localhost:8320
