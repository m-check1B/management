# mail-tba Decision — 2026-04-29

## Decision: Keep mail-tba as dedicated mail + vault host

### Rationale
1. **Mail-in-a-Box is complex and working.** Migrating Postfix/Dovecot/DNS/SpamAssassin is high-risk with zero user-facing benefit. It handles MX records, DKIM, SPF, DMARC, spam filtering, and IMAP for 3 domains. One misconfiguration = lost email.

2. **Mail needs its own IP.** Reverse DNS (PTR record) for mail deliverability is tied to the IP. Sharing an IP with other services risks blacklisting. A dedicated mail host is industry best practice.

3. **Infisical is a perfect fit.** Lightweight (3 containers), already running, already has DNS + TLS at vault.kraliki.com. Zero reason to move it.

4. **Ubuntu 22.04 is fine for now.** The rebuild to 26.04 was aspirational. The server is stable, has 91 days uptime, and works. Don't fix what isn't broken.

### What stays on mail-tba
| Service | Why |
|---------|-----|
| Mail-in-a-Box | Critical business infrastructure, complex to migrate |
| Infisical (vault.kraliki.com) | Lightweight, already configured |
| Traefik | Routes vault.kraliki.com and websites |
| ZeroTier | Network mesh |

### What will eventually move off
| Service | Destination | When |
|---------|-------------|------|
| 3 static websites | Cloudflare Pages | When ready to consolidate |
| Cal.com | prod-control-plane | Next migration batch |
| Kiki API | prod-control-plane | Next migration batch |

### Rename plan (deferred)
- Current hostname: `mail.kraliki.com` (set by Mail-in-a-Box)
- Proposed: `kraliki-mail` or keep as-is (mail.kraliki.com is correct for a mail server)
- **Do NOT rebuild on 26.04** — too risky for working mail infrastructure

### DR strategy
- mail-tba becomes the **mail DR target**: nightly backups of prod-control-plane DBs
- Already has encrypted preflight DB dumps from earlier
- Add automated backup sync from prod VMs → mail-tba storage

### Anti-patterns avoided
- ❌ Don't mix mail with production app hosting (deliverability risk)
- ❌ Don't rebuild a working mail server for OS freshness
- ❌ Don't migrate Mail-in-a-Box (it's a monolithic appliance)
- ❌ Don't put mail on the same IP as web apps (blacklist risk)

## Source
Matej: "as for mailserver, i am not sure about it" → "you decide what is best"
TBA-One-PA decision: keep it, protect it, use it.
