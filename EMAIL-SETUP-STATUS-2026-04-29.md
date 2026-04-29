# Email Setup Status — 2026-04-29

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INBOUND EMAIL                                │
│                                                                      │
│  kraliki.com ─── Cloudflare Email Routing (FREE) ──── catch-all ──┐ │
│  verduona.com ── Cloudflare Email Routing (FREE) ──── catch-all ──┤ │
│  perfectmission.co.uk ── Cloudflare Email Routing ── catch-all ──┤ │
│                                                                    │
│  All forward to: matej.havlin@protonmail.com ◄─────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         OUTBOUND EMAIL                               │
│                                                                      │
│  councilnow.com ─── Resend API (FREE, 1 domain) ─── transactional   │
│  (verification emails, notifications, etc.)                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         SEND-AS (ProtonMail Pro)                     │
│                                                                      │
│  matej.havlin@protonmail.com (primary)                               │
│  ├── matej@verduona.com (send-as)                                    │
│  ├── matej@kraliki.com (send-as)                                     │
│  ├── matej@perfectmission.co.uk (send-as)                            │
│  └── any@* (catch-all catches replies)                               │
└─────────────────────────────────────────────────────────────────────┘
```

## Cloudflare Email Routing Status

| Domain | Status | DNS | Catch-All | Destination |
|--------|--------|-----|-----------|-------------|
| kraliki.com | ✅ Enabled | ✅ Locked | ⏳ Pending verification | matej.havlin@protonmail.com |
| verduona.com | ✅ Enabled | ✅ Locked | ⏳ Pending verification | matej.havlin@protonmail.com |
| perfectmission.co.uk | ✅ Enabled | ✅ Locked | ⏳ Pending verification | matej.havlin@protonmail.com |

### DNS Records (per domain)
- **MX**: route1/2/3.mx.cloudflare.net (locked)
- **SPF**: `v=spf1 include:_spf.mx.cloudflare.net ~all` (locked)
- **DKIM**: cf2024-1._domainkey (locked)

### Destination Address
- **Email**: `matej.havlin@protonmail.com`
- **Status**: ⏳ PENDING — verification email sent, waiting for Matej to click link
- **Created**: ~23 minutes ago
- **Verification**: Account-wide (once verified, works for all zones)

## Resend Status

| Domain | Status | Sending | Plan |
|--------|--------|---------|------|
| councilnow.com | ✅ verified | ✅ enabled | Free (1 domain limit) |

- **API Key**: stored at `/Users/matejhavlin/github/secrets/resend_api_key.txt`
- **Region**: us-east-1

## What's Done

1. ✅ kraliki.com Email Routing enabled (old MX + SPF deleted, new locked DNS created)
2. ✅ verduona.com Email Routing enabled (old MX + SPF deleted, new locked DNS created)
3. ✅ perfectmission.co.uk Email Routing enabled (old MX + SPF deleted, new locked DNS created)
4. ✅ Destination address added (`matej.havlin@protonmail.com`)
5. ✅ Verification email sent (and resent)
6. ✅ Resend `councilnow.com` verified and sending enabled

## What Matej Needs To Do

### 1. Verify Destination Email (BLOCKING)
- Check ProtonMail inbox for email from Cloudflare
- Click the verification link
- This unblocks ALL catch-all rules for ALL domains

### 2. Configure ProtonMail Send-As
Matej has ProtonMail Pro. He needs to add send-as identities:
- Log into ProtonMail → Settings → Identity → Addresses
- Add each business address:
  - `matej@verduona.com`
  - `matej@kraliki.com`
  - `matej@perfectmission.co.uk`
- ProtonMail will send verification emails to those addresses
- Since Cloudflare catch-all forwards to ProtonMail, the verification emails will arrive in the same inbox
- Click each verification link to activate send-as

### 3. Test the Flow
After everything is set up:
- Send a test email TO `test@kraliki.com` → should arrive in ProtonMail
- Reply FROM ProtonMail using `matej@kraliki.com` send-as
- Verify the round-trip works

## Business Email Addresses

Once set up, Matej can receive at and send from:

| Address | Purpose |
|---------|---------|
| matej@kraliki.com | Platform/business |
| matej@verduona.com | Verduona brand |
| matej@perfectmission.co.uk | PerfectMission |
| anything@kraliki.com | Catch-all (all mail) |
| anything@verduona.com | Catch-all (all mail) |
| anything@perfectmission.co.uk | Catch-all (all mail) |

## Notes
- Destination address verification is **account-wide** — once verified for one zone, it's verified for all
- Cloudflare Email Routing is **free** — no limits on addresses or domains
- Resend free plan = 1 domain, 100 emails/day, 3000/month — enough for councilnow.com transactional
- Automation/system emails (from AgentJack, CouncilNow, etc.) use Resend, not ProtonMail
- ProtonMail is for Matej's direct business correspondence only
