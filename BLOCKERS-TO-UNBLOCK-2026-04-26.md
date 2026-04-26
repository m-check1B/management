# Blockers to Unblock — 2026-04-26 06:56 CEST

This is the single current unblock list. Source of truth checked: `SPRINT-CURRENT.md`, `COUNCILNOW-REVENUE-AUDIT.md`, Cloudflare Phase 1 plan, sales packet, and active Linear issues.

## Top founder / human unblock actions

### 1) AgentJack Telegram continuity proof
- **Status:** Runtime/ownership is healthy; final red check is inbound continuity.
- **Current recorded blocker:** `telegram.inboundContinuityVerified=false` until a Telegram reply lands containing the current token recorded in Sprint: `AJ-ED58ED`.
- **What Matej can do:** Reply in Telegram to `@matejs_agent_jack_bot` with the token `AJ-ED58ED`.
- **If token is stale:** ask TBA to generate a fresh AgentJack delivery token and re-run live validation.
- **Why it matters:** This flips the canonical AgentJack `ag-webui` + runtime lane from “healthy but unproven inbound” to green.
- **Source:** `SPRINT-CURRENT.md` Blocked section.

### 2) CouncilNow first-customer outreach approval
- **Status:** CouncilNow is now **ready for first customer**. Core customer path is green.
- **Blocked on:** Founder approval before any external outreach/send.
- **What Matej can do:** approve one of:
  1. “Send to first 10 targets using current Decision Sprint offer.”
  2. “Prepare targets/drafts only, no send.”
  3. “Use controlled pilot wording with limitations disclosed.”
- **Linear:** `KRA-3673` — `[Sales][P0] CouncilNow first customer outreach packet`.
- **Proof:** production auth→checkout, product smoke, share/public API, analytics isolation all passed; latest checkout proof refreshed at 04:35 and another E2E passed at 06:36.

### 3) Cloudflare website cutover / Kiki API parity
- **Status:** DNS cutover is paused.
- **Blocked on:** `/api/kiki/*` parity. Live domains return Kiki API JSON; Cloudflare Pages targets currently return HTML fallback.
- **What Matej can unblock/decide:** confirm preferred stable backend hostname, default recommendation: `kiki-api.verduona.dev` on dev-2026.
- **What TBA/Axis should do next:**
  1. verify/stand up Kiki backend container on dev-2026,
  2. expose stable Traefik origin,
  3. add Cloudflare Pages `/api/kiki/*` proxy,
  4. verify JSON parity on both Pages projects,
  5. only then cut DNS.
- **Linear:** `KRA-3672` — `[Infra][P0] Serve Kiki API from dev-2026 backend and proxy /api/kiki from Cloudflare Pages`.
- **Source:** `INFRA-MOVE-PHASE1-CLOUDFLARE-SITES.md`.

### 4) AgentJack voice route guard decision
- **Status:** In Review.
- **Blocked on:** explicit policy decision for AG WebUI voice routes: route guard/security policy vs intended live voice/transcribe behavior.
- **What Matej can decide:** should `chat/transcribe` and `chat/live-voice/session` require the same auth guard as other write routes, or are they allowed under a narrower voice-session guard?
- **Linear:** `KRA-3666` — `[AgentJack][P1] Fix AG WebUI voice API regressions`.

you decide

### 5) OpenClaw / specialist model-routing upgrade approval
- **Status:** quality/model-routing audit exists; live config not patched.
- **Blocked on:** explicit approval because it changes OpenClaw config and requires gateway restart + doctor check.
- **What Matej can approve:** “Patch model routing per quality audit, restart gateway, run `openclaw doctor --non-interactive`.”
- **Linear:** `KRA-3671` — `[Ops][P1] Fix JSON/text formatting quality gate and update specialist model routing`.
- **Why it matters:** prevents JSON/metadata leakage into human copy and upgrades high-leverage specialist lanes away from weaker defaults.

yes

### 6) Founder mail routing to Proton
- **Status:** Backlog, ready to execute carefully.
- **Blocked on:** mailbox/DNS provider flow and founder approval before mail DNS changes.
- **What Matej can unblock:** confirm Proton destination and allow DNS work for:
  - `matej@verduona.com`
  - `matej@kraliki.com`
  - `perfectmission.co.uk` founder mail
- **Linear:** `KRA-3669` — `[Infra][P1] Route founder business mail into Proton handling`.
- **Guardrail:** do not touch app/product transactional email; that stays Resend.

approved

### 7) Self-hosted mail migration decision
- **Status:** Evaluation backlog.
- **Blocked on:** decision after assessment: keep self-hosted mail, migrate it to dev-2026, or retire/standby it.
- **What Matej can unblock:** approve evaluation only, or defer until Proton/Resend split is complete.
- **Linear:** `KRA-3670` — `[Infra][P1] Evaluate moving self-hosted mail server from dedicated VM to dev-server`.

do it today

### 8) Resend sender-domain verification for CouncilNow emails
- **Status:** Non-blocking for first customer; product works in-app.
- **Blocked on:** verifying `councilnow.com` sender domain in Resend or choosing a verified sender domain.
- **What Matej can unblock:** approve DNS verification work for Resend sender domain.
- **Linear:** `KRA-3679` — `[CouncilNow][P2] Verify Resend sender domain for report-ready emails`.

you need end to and loop control

## TBA/Axis execution backlog — not necessarily founder-blocked

### 9) Website naming alignment
- **Linear:** `KRA-3667` — `[Websites][P1] Surgical naming alignment for kraliki.com + verduona.com to current lineup`.
- **Blocker:** mostly execution time; can be done surgically.

must be done today

### 10) CouncilNow cross-reference on websites
- **Linear:** `KRA-3668` — `[Websites][P1] Add CouncilNow cross-reference to kraliki.com + verduona.com`.
- **Blocker:** mostly execution time; should be coordinated with naming alignment and Cloudflare Pages state.

must be done today

## Cleared / no longer blockers

- CouncilNow Stripe price ID bug — fixed.
- CouncilNow auth→checkout E2E — repeatedly green, latest 04:35 and 06:36 proof passed.
- CouncilNow first-customer product blockers — fixed/closed: `KRA-3675`, `KRA-3676`, `KRA-3677`, `KRA-3678`.
- CouncilNow is first-customer-ready; remaining blocker is **send approval**, not product readiness.
- Cron health — no enabled job has `consecutiveErrors > 2`; no overdue non-running jobs in latest heartbeat/selfcheck.

is the jason formating problem insome reports fixed? are the models and promt ptimized as we have new models that we aldedy use in our openclaw and council shuold be rething on models.


## Recommended unblock order

1. Reply to AgentJack Telegram token or ask TBA for a fresh one. - I asked
2. Approve CouncilNow first-customer outreach prep/send path. - you can split, fix move the emails and then do it, but you need to have one dummy customer so you can end to end test
3. Approve Kiki stable origin (`kiki-api.verduona.dev`) so Cloudflare cutover can proceed. fine with me.
4. Decide AG WebUI voice route guard policy. you decide thats just technicality
5. Approve model-routing/config patch if you want quality gates upgraded now. you decide thats just technicality
6. Then handle Proton mail + Resend sender verification. i would start with this.
