# Mail / DNS Execution Lane — 2026-04-26

_Status: Resend DNS records added and verification loop running; Proton founder mail blocked on Proton UI-generated records_

## Scope
1. Proton founder mail routing (`verduona.com`, `kraliki.com`, `perfectmission.co.uk`)
2. Resend sender-domain verification (`councilnow.com`)
3. Self-hosted mail server evaluation (`mail-tba` / `mail.kraliki.com`)

---

## 1. Resend Sender Domain — councilnow.com

### Done
- Resend domain `councilnow.com` created via API:
  - `id=21b7a997-7224-46d3-9067-6e565b3db698`
- Added required DNS records in Cloudflare zone `councilnow.com` via the logged-in Cloudflare dashboard session.

### DNS records added

| Type | Name | Value | Priority |
|------|------|-------|----------|
| TXT | `resend._domainkey` | `p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDblx4tM1+xad6cMB5qBCQNgIjoEQlszYMll2xWajcso8U8wwMKC+GHfPwOSqOYpmBvHYhDfIQ6X+LlA7hWT1jeURt9a/+70S+JuccCVEHBiQKpUFX1xQspoR35GpVy2ee6zLTg8psJVOOoUayKKYBo4gqdNA88MghurRqVcdyNjwIDAQAB` | — |
| MX | `send` | `feedback-smtp.us-east-1.amazonses.com` | 10 |
| TXT | `send` | `v=spf1 include:amazonses.com ~all` | — |

### Verification evidence

```bash
dig +short TXT resend._domainkey.councilnow.com @1.1.1.1
# present

dig +short MX send.councilnow.com @1.1.1.1
# 10 feedback-smtp.us-east-1.amazonses.com.

dig +short TXT send.councilnow.com @1.1.1.1
# "v=spf1 include:amazonses.com ~all"
```

Triggered Resend verification:

```bash
curl -X POST https://api.resend.com/domains/21b7a997-7224-46d3-9067-6e565b3db698/verify
# returned id=21b7a997-7224-46d3-9067-6e565b3db698
```

Current Resend status after initial verification attempt:

```json
{
  "id": "21b7a997-7224-46d3-9067-6e565b3db698",
  "name": "councilnow.com",
  "status": "pending",
  "region": "us-east-1"
}
```

### Follow-up evidence — 2026-04-26 06:54 UTC / 08:54 Europe/Prague

Resend API GET before re-trigger:

```json
{
  "id": "21b7a997-7224-46d3-9067-6e565b3db698",
  "name": "councilnow.com",
  "status": "pending",
  "region": "us-east-1",
  "records": [
    { "record": "DKIM", "name": "resend._domainkey", "type": "TXT", "status": "pending" },
    { "record": "SPF", "name": "send", "type": "MX", "status": "verified", "value": "feedback-smtp.us-east-1.amazonses.com", "priority": 10 },
    { "record": "SPF", "name": "send", "type": "TXT", "status": "verified", "value": "v=spf1 include:amazonses.com ~all" }
  ]
}
```

Public DNS evidence from `1.1.1.1`, `8.8.8.8`, and `9.9.9.9`:

```bash
TXT resend._domainkey.councilnow.com
# present; matches Resend DKIM value

MX send.councilnow.com
# 10 feedback-smtp.us-east-1.amazonses.com.

TXT send.councilnow.com
# "v=spf1 include:amazonses.com ~all"
```

Resend verification was triggered again:

```bash
POST /domains/21b7a997-7224-46d3-9067-6e565b3db698/verify
# {"object":"domain","id":"21b7a997-7224-46d3-9067-6e565b3db698"}
```

Refreshed Resend status immediately after re-trigger:

```json
{
  "id": "21b7a997-7224-46d3-9067-6e565b3db698",
  "name": "councilnow.com",
  "status": "pending",
  "region": "us-east-1",
  "records": [
    { "record": "DKIM", "name": "resend._domainkey", "type": "TXT", "status": "pending" },
    { "record": "SPF", "name": "send", "type": "MX", "status": "pending" },
    { "record": "SPF", "name": "send", "type": "TXT", "status": "pending" }
  ]
}
```

### Next step
- DNS looks correct on public resolvers, but Resend still reports `pending`.
- One final cron follow-up is scheduled for 2026-04-26 07:28:19 UTC / 09:28:19 Europe/Prague (`id=26cc2e53-648d-42a4-be01-7fd979703655`); if it is still pending then, report the pending state instead of looping indefinitely.

---

## 2. Proton Founder Mail Routing

### Domains and current state

| Domain | Current MX | Current SPF | Proton status |
|--------|------------|-------------|---------------|
| verduona.com | `10 mail.kraliki.com` | `v=spf1 mx -all` | Not set up |
| kraliki.com | `10 mail.kraliki.com` | `v=spf1 mx -all` | Not set up |
| perfectmission.co.uk | `10 mail.kraliki.com` | `v=spf1 mx -all` | Not set up |

All three domains currently route mail to the self-hosted Mail-in-a-Box on `mail-tba` (`91.99.176.2`).

### Proton Mail required records per domain

| Type | Name | Value | Priority |
|------|------|-------|----------|
| MX | `@` | `mailsec.protonmail.ch` | 10 |
| MX | `@` | `mail.protonmail.ch` | 20 |
| TXT | `@` | `v=spf1 include:_spf.protonmail.ch ~all` | — |
| CNAME | `protonmail._domainkey` | Proton-provided per-domain DKIM value | — |
| TXT | `@` | Proton-provided verification code | — |

### Blocker
Proton requires interactive domain setup in the Proton Mail UI to generate:
1. Domain verification TXT record
2. Per-domain DKIM CNAME record

Those values are not derivable safely from DNS/API alone.

### Next step for founder / UI
Open Proton Mail → Settings → Domains → add:
- `verduona.com`
- `kraliki.com`
- `perfectmission.co.uk`

Then either paste the Proton verification TXT + DKIM records here, or leave the browser session open on the records screen and I can copy/apply them.

---

## 3. Self-Hosted Mail Server Evaluation — mail-tba

### Current state
- Host: `mail-tba` (`91.99.176.2`), hostname `mail.kraliki.com`
- Software: Mail-in-a-Box with Postfix + Dovecot
- Uptime observed: 87 days
- Disk observed: 25G used / 75G total (34%)
- Mail queue: empty
- DR role: already designated as DR standby for dev-2026 per `DR-STANDBY-PLAN-MAIL-TBA.md`
  - Tier 0 encrypted backups running every 15 min
  - Tier 1 hourly, Tier 2/3 daily

### Recommendation

**Standby → retire mail after Proton cutover.**

| Option | Judgment |
|--------|----------|
| Keep as-is | Works, but wastes a dedicated VM on mail after Proton cutover |
| Migrate MIAB to dev-2026 | Bad fit: MIAB wants its own IP/domain and adds service conflicts |
| Retire mail services after Proton | Best: keep VM useful as DR-only, stop mail stack once MX leaves it |
| Immediate shutdown | Not safe until Proton MX is live |

### Action plan
1. Keep `mail-tba` running until Proton MX cutover completes.
2. After Proton is verified, stop Postfix/Dovecot/MIAB services.
3. Keep the VM as DR standby.
4. Later evaluate shrinking VM size.

---

## Summary

| Item | Status | Next |
|------|--------|------|
| Resend DNS | Added in Cloudflare | Wait for Resend verification to flip from `pending` |
| Resend verification | Triggered | Cron follow-up scheduled |
| Proton founder mail | Blocked | Need Proton UI-generated TXT/DKIM values |
| Self-hosted mail | Evaluated | Keep until Proton cutover, then retire mail services / keep DR |

## Files/tools inspected
- `/Users/matejhavlin/github/management/DR-STANDBY-PLAN-MAIL-TBA.md`
- `/Users/matejhavlin/github/management/INFRA-MAP.md`
- `/Users/matejhavlin/github/management/BLOCKERS-TO-UNBLOCK-2026-04-26.md`
- Cloudflare dashboard/API via logged-in browser session
- Resend API
- SSH to `mail-tba`
- SSH to `dev-2026`
- `~/.ssh/config`
