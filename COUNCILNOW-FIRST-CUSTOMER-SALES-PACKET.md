# CouncilNow — First Customer Sales Packet

_Last updated: 2026-04-26 00:22 CET_

## Current status
CouncilNow is **ready for first customer**.

Production proof has repeatedly passed:
- registration
- login
- `/auth/me`
- subscription status
- Stripe billing portal
- checkout for starter/pro/scale/enterprise
- Stripe checkout URL `HEAD 200`

## Business objective
Convert the first real customer without waiting for more product work.

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

## Next internal actions
1. Pick first 20 target names.
2. Draft 3 outreach variants.
3. Run final quality/model gate.
4. Get founder approval before any external send.
5. Track responses and objections.
