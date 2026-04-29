# mail-tba Rename + Ubuntu 26.04 LTS Plan

_Date: 2026-04-29 09:31 Europe/Prague_  
_Status: planned; no hostname/DNS/OS change executed_

## Decision

`mail-tba` is a misleading name for the end state. It should be renamed and rebuilt/upgraded to Ubuntu 26.04 LTS as part of the Kraliki production infra cleanup.

Current read-only fact check on 2026-04-29:

- SSH target/IP: `91.99.176.2`
- Current hostname: `mail.kraliki.com`
- Current OS: `Ubuntu 22.04.5 LTS` (`jammy`)
- Current kernel: `5.15.0-164-generic`

## Proposed end-state name

Use `kraliki-dr-standby` as the internal/infrastructure name unless Matej chooses another.

Rationale:

- The VM should no longer be conceptually “mail”.
- Its target role is active-passive DR / backup / restore-validation standby.
- Public mail should move to Resend + Proton Mail, and public product services should move to `dev-2026` production VMs or Cloudflare Pages.

## Safety note

Do **not** run an in-place OS upgrade on the live `mail-tba` while it still serves mail/web/CRM/backup responsibilities.

Safer path:

1. Evacuate or explicitly preserve each current responsibility.
2. Verify backups/restores off-host.
3. Rebuild or replace the VM on Ubuntu 26.04 LTS.
4. Reattach only the DR/standby responsibilities.
5. Rename internal host labels/aliases after public dependencies are removed or mapped.

Reason: the box is currently Ubuntu 22.04 and likely carries Mail-in-a-Box/web/CRM assumptions. A direct 22.04 → 24.04 → 26.04 upgrade while live could break mail/web services and backup landing paths.

## Migration phases

### Phase A — inventory and evacuation map

Inventory current `mail-tba` responsibilities:

- `kraliki.com` / `www.kraliki.com` marketing root
- `mail.kraliki.com` / Mail-in-a-Box
- `crm.kraliki.com` / Twenty CRM
- DR backup landing paths under `/home/adminmatej/dr-backups`
- any residual `*.verduona.dev` wildcard/catch-all traffic
- any Infisical/secrets/vault service still depending on the VM

Output: a current service/process/compose/path inventory with secrets redacted.

### Phase B — move public non-DR roles away

Target moves:

- Static marketing → Cloudflare Pages or other static deploy target.
- Transactional/app outbound mail → Resend.
- Human mailbox/ops mail → Proton Mail.
- CRM either stays only if explicitly desired or moves to a better named ops/CRM home.
- Unknown wildcard `*.verduona.dev` drift is removed only after explicit host mapping is complete.

### Phase C — protect DR data

Before rebuild/replace:

- Verify latest DB backup restore on a non-`mail-tba` target.
- Verify AgentJack state backup restore on a non-`mail-tba` target.
- Copy current `/home/adminmatej/dr-backups` off-box or ensure Storage Box/offsite copy exists.
- Confirm `dev-2026` backup jobs can target the replacement path.

### Phase D — rebuild/replace on Ubuntu 26.04 LTS

Preferred implementation:

- Create a fresh Ubuntu 26.04 LTS replacement VM or rebuild this VM only after public roles are evacuated.
- Set host/internal name to `kraliki-dr-standby` or final chosen name.
- Install only DR/restore validation baseline:
  - SSH hardened access
  - backup landing directories
  - restore-validation tooling
  - monitoring/health checks
  - optional warm standby DB containers after restore automation is green

### Phase E — cutover backup landing path

- Update `dev-2026` backup rsync targets from `mail-tba:` to the final alias/name.
- Run DB backup + restore validation.
- Run AgentJack state backup + restore validation.
- Record proof artifacts.

## Explicit non-goals before checkpoint

- No DNS changes yet.
- No mail MX changes yet.
- No CRM migration yet.
- No OS upgrade or reboot of the live `mail-tba` yet.
- No deletion of existing backup artifacts.

## Acceptance criteria

- New name is reflected in docs, SSH aliases, backup scripts, and monitoring.
- Ubuntu 26.04 LTS verified on the standby host.
- Restore validation passes for Tier 0 DBs and AgentJack state.
- No public product or mail service depends on the old `mail-tba` naming/model.
- Rollback path exists until backups are proven on the renamed/rebuilt standby.
