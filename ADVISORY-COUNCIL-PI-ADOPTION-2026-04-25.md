## Advisory Council: Should we adopt Pi-style solo-purpose specialist agents as the default operating model between TBA-One-PA (Mac) and Axis (dev-2026)?

### Grounding Context
- Current control-plane model is already split: Mac orchestration + dev-2026 execution.
- Linear queue is trimmed to 4 active AgentJack issues (KRA-3663..3666).
- Health baseline is stable (PM2 online 14, cron errors >2 = 0, CouncilNow billing checks pass repeatedly).
- Existing fleet already has specialist identities (`coder`, `codex-reviewer`, `kimi-frontend`, `claude-infra`, `grok-researcher`, `claude-copy`), but packet discipline has been inconsistent.

### Moderator Framing (Issue Tree)
- **Option A (Pi-style default):** strict single-purpose packets, reviewer chain, evidence-first completion.
- **Option B (mixed-role default):** keep broad horizontal prompts where one run may implement/review/deploy/report.

## Individual Takes

## Round 1 — Predictions

**The Strategist**
- Option A prediction: execution throughput improves in 2–4 weeks because context collision drops. Expected +20–35% first-pass acceptance on P0/P1 tickets.
- Option B prediction: short-term feels faster, but 30–60 day drift increases (more rework loops, less deterministic quality).
- Key variable: enforcement consistency by TBA-One-PA.
- Hidden assumption: team keeps one live priority lane.
- Confidence: 8/10.

**The Pragmatist**
- Option A prediction: Monday-morning execution becomes clearer immediately: one lane, one packet, one proof gate. Time-to-evidence should drop from ~90 minutes to ~45–60 minutes per packet.
- Option B prediction: continuing mixed packets preserves ambiguity and causes “done but not verified” churn.
- Key variable: quality of acceptance checks in packet headers.
- Hidden assumption: packet template is used every time.
- Confidence: 8/10.

**The Contrarian**
- Option A prediction: could over-fragment work and create handoff tax if packet boundaries are too narrow; net gain only if escalation rules are strict.
- Option B prediction: for tiny fixes (<15 minutes), mixed-mode can be faster with less orchestration overhead.
- Key variable: minimum packet size threshold.
- Hidden assumption: reviewer capacity is available when needed.
- Confidence: 6/10.

**The Customer**
- Option A prediction: customer-facing reliability improves (fewer regressions in auth/billing/voice paths), which improves trust and conversion. A 5–10% improvement in perceived reliability often beats feature count.
- Option B prediction: users keep seeing uneven quality spikes; confidence in “stable B2B” claim weakens.
- Key variable: escaped regressions after “done”.
- Hidden assumption: reliability signals are visible in product behavior.
- Confidence: 7/10.

**The Operator**
- Option A prediction: operations load gets healthier because incident ownership is explicit per lane; MTTR for runtime issues should reduce by ~25% if infra lane is isolated.
- Option B prediction: mixed-role runs blur ownership and slow root-cause isolation.
- Key variable: whether evidence artifacts are mandatory before handoff closure.
- Hidden assumption: no bypasses for “just this once” mixed packets.
- Confidence: 8/10.

**The Analyst**
- Option A prediction: measurable KPI gains in 7 days: lower rework loops, lower token burn per completed issue, higher first-pass closure. Expected net: +15–30% efficiency if discipline holds.
- Option B prediction: metrics stay noisy; difficult to attribute wins/failures because one run does everything.
- Key variable: metric instrumentation quality.
- Hidden assumption: baseline metrics are captured from day 1.
- Confidence: 7/10.

## Round 2 — Stress-Test Predictions (Debate / Cross-Examination)

**Contrarian → Strategist:** “Your +20–35% first-pass gain is optimistic. What if handoff tax erases that? At small scale, extra packet ceremony can cost 10–20 minutes each.”

**Strategist responds:** Fair point. The prediction is wrong if packet size is too small. I adjust: gains hold only when packets target >30-minute tasks or P0/P1 risk domains.

**Pragmatist → Contrarian:** “Your ‘tiny fix exception’ is valid, but mixed-mode tends to leak into larger work. The real risk nobody talks about is exception creep.”

**Contrarian responds:** I hear you on exception creep. Add a hard threshold: mixed-mode allowed only for low-risk, single-file, no-runtime-touch changes.

**Operator → Pragmatist:** “Your 45–60 minute time-to-evidence assumes good templates. If packet quality is weak, we lose time arguing scope.”

**Pragmatist responds:** Agreed. Here’s what to do: enforce mandatory packet fields and reject packets missing exact acceptance checks.

**Customer → Analyst:** “You predict +15–30% efficiency, but customers care about reliability, not internal speed. Show how that converts to trust.”

**Analyst responds:** Good point. We should track escaped regressions/week and correlate with trial-to-paid conversion over 30 days.

**Strategist → Operator:** “Your MTTR reduction assumes lane purity. What if infra + app bugs are intertwined?”

**Operator responds:** Then lane purity still helps: infra lane owns environment proof, app lane owns code proof. Joint incidents can be split with explicit ownership.

**Analyst → Contrarian:** “You’re right about handoff tax, but your estimate ignores rework savings. The alternative view is that 1 prevented rollback can save 2–4 hours.”

**Contrarian responds:** Concede that point. For high-blast-radius paths (auth, billing, runtime), specialist discipline likely wins net time.

## Round 3 — Updated Predictions (After Challenges)

**The Strategist (updated)**
- Revised prediction: Option A still wins, but expected gain narrows to +15–25% in first-pass acceptance unless packet-quality enforcement is strict.
- Confidence update: **8/10 → 8/10** (same confidence, tighter condition).

**The Pragmatist (updated)**
- Revised prediction: Option A wins immediately if packet rejection rule is enforced on missing acceptance checks.
- Confidence update: **8/10 → 9/10** after Operator challenge on template quality.

**The Contrarian (updated)**
- Revised prediction: Option A is better for most strategic/technical work; Option B should exist only as a tiny-fix exception policy.
- Confidence update: **6/10 → 7/10** after Analyst and Pragmatist pushback.

**The Customer (updated)**
- Revised prediction: Option A likely improves customer trust if reliability metrics are published internally and used for release gates.
- Confidence update: **7/10 → 8/10** after correlation proposal.

**The Operator (updated)**
- Revised prediction: Option A reduces incident ambiguity and should cut MTTR if evidence artifacts are non-negotiable.
- Confidence update: **8/10 → 8/10**.

**The Analyst (updated)**
- Revised prediction: Option A expected net positive within 7 days, with strongest effect on rework loops and escaped regressions.
- Confidence update: **7/10 → 8/10**.

## Synthesis

### Option A predicted outcome
- Higher execution reliability, cleaner ownership, measurable efficiency gains in 7 days.
- Expected KPI direction: first-pass acceptance up, rework loops down, escaped regressions down.

### Option B predicted outcome
- Occasional speed wins on tiny tasks, but rising ambiguity and lower predictability on high-risk work.

### Recommendation
**Adopt Option A as default now** (Pi-style specialist packets), with a tightly scoped tiny-fix exception.

### Confidence
**8.3/10**

### Key variable
Strict packet discipline by TBA-One-PA (objective, scope, acceptance checks, evidence, escalation).

### Key risk / failure mode
- Exception creep (“just this once” mixed runs) erodes the model.
- Weak packet definitions create coordination overhead.

### Dissent
- Contrarian dissent remains: mixed-mode may outperform for low-risk, sub-15-minute edits.
- Council accepts dissent and encodes it as a narrowly bounded exception, not default behavior.

### Next steps (actionable)
1. Enforce packet template on all KRA-3663..3666 work this week (start today).
2. Add tiny-fix exception rule: single-file, no runtime touch, <15 minutes, no external side effects.
3. Track 5 pilot metrics daily for 7 days: first-pass acceptance, rework loops, time-to-evidence, token/cost per completed issue, escaped regressions.
4. Require reviewer lane sign-off for auth/billing/runtime/CI changes before closure.
5. Re-evaluate after 7 days and either keep, tighten, or rollback exception policy.

### Final statement
Prediction favors Pi-style specialist operation as the default coordination policy, with disciplined exception boundaries.
