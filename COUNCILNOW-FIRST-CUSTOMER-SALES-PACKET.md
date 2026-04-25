# CouncilNow — First Customer Sales Packet

_Last updated: 2026-04-26 00:27 CET_

## Current status
CouncilNow is **billing-path verified** and sales-prep can continue, but it is **not yet fully first-customer-ready** until remaining product-quality bugs are triaged and either fixed or explicitly accepted for a controlled pilot.

Production billing proof has repeatedly passed:
- registration
- login
- `/auth/me`
- subscription status
- Stripe billing portal
- checkout for starter/pro/scale/enterprise
- Stripe checkout URL `HEAD 200`

## Business objective
Prepare first-customer acquisition while closing the quality gate. Do not send external outreach or claim full readiness until bug triage is complete and founder approves the send.

## Primary ICP for first 10 conversations
1. Founders / CEOs making strategic product, pricing, hiring, or market-entry decisions.
2. High-agency managers with recurring complex decisions and budget authority.
3. Vibe coders / AI builders deciding what to build, kill, or prioritize.
4. Trading / investing operators who need structured decision debate and post-decision audit trails.

## Offer v1
**CouncilNow Decision Sprint**

A structured multi-agent decision session for one important business decision, with:
- decision framing
- multiple expert perspectives
- debate + disagreement surfacing
- recommendation summary
- next-step plan
- optional follow-up review

## Price ladder
- Self-serve SaaS tier: existing public pricing path.
- Founder-led first-customer Sprint: **€299–€999** depending on complexity.
- Business/team monthly plan: use public Pro/Scale/Enterprise tiers after first proof.

## First-customer positioning
Not “AI chat”.

Position as:
> A boardroom-grade decision sparring system for founders and executives who need better decisions faster.

## Outreach angle
Subject/DM opener:
> Quick idea for your next hard decision

Message:
> I’m piloting CouncilNow: a multi-agent decision sprint for founders/executives. You bring one real strategic decision; the system pressure-tests it from multiple expert angles and gives you a clear recommendation + next-step plan. The product is live, including checkout. Want to try one decision sprint this week?

## Call script
1. “What decision are you currently postponing or debating?”
2. “What makes it high-stakes?”
3. “Who disagrees internally, and why?”
4. “What would a good answer unlock?”
5. “If we could pressure-test this in 45 minutes and give you a board-ready summary, would that be useful?”

## Quality gate before sending any customer-facing artifact
- No raw JSON, metadata envelopes, schema fragments, or tool-state leakage.
- Markdown/plain text must render cleanly.
- The offer must be concrete: who, what decision, what outcome, what next step.
- Use premium model routing for final copy; do not use weak/default mini models for final customer-facing output.

## Quality/readiness gate before first outreach
1. List known bugs and classify: launch-blocker / pilot-acceptable / cosmetic.
2. Re-test core customer path beyond billing: landing page → signup/login → decision/session creation → report/output → billing/portal.
3. Fix launch-blockers or document pilot limitations explicitly.
4. Run final formatting/model-quality gate.
5. Get founder approval before any external send.

## Next internal actions
1. Build bug triage list.
2. Pick first 20 target names in parallel, but keep outreach unsent.
3. Draft 3 outreach variants.
4. Run final quality/model gate.
5. Track responses and objections only after founder-approved send.
