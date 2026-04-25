# Quality + Model Routing Audit — 2026-04-26

## Founder signal
Matej flagged quality concerns:
- JSON leaking / converting badly into text formatting.
- Output quality may be uneven.
- Specialist lanes should have better model selection and be kept current as newer stronger models appear.

## Diagnosis

### 1) JSON → text formatting bug class
Problem pattern:
- Internal structured payloads, metadata, or machine JSON can leak into user-facing prose or markdown.
- This makes business/customer artifacts feel unfinished.

Required rule:
- Keep machine JSON and final human-facing output separate.
- Only show JSON when explicitly requested.
- Every business/customer artifact needs a final readability/formatting gate.

Acceptance check:
- No raw metadata envelopes in final customer/business copy.
- Markdown renders cleanly.
- Tables/lists are human-readable.
- CTAs and next steps are plain language, not schema language.

### 2) Model-routing drift
Current Mac OpenClaw agent map observed:

| Agent | Current model | Concern |
|---|---|---|
| `main` | `openai-codex/gpt-5.3-codex` | Good orchestrator, but Business Director / high-stakes strategy may need GPT-5.5 lane. |
| `marketing-director` | `openai-codex/gpt-5.4-mini` | Too weak for premium positioning/strategy artifacts. |
| `claude-copy` | `kimi-cli/kimi-for-coding` | Coding model is not ideal default for premium copy. |
| `grok-researcher` | `openrouter/x-ai/grok-4.20-beta` | Should align to configured current xAI aliases where possible. |
| `claude-infra` | `zai/glm-5-turbo` | Acceptable fast infra lane, but high-risk infra needs stronger model escalation. |
| `coder` / `codex-reviewer` | `codex-cli/gpt-5.4` | Strong code lane; keep, with GPT-5.5 escalation for architecture/security decisions. |

## Recommended routing map

| Lane | Default | Escalation |
|---|---|---|
| Business Director / founder strategy | `openai-codex/gpt-5.5` | `openai-codex/gpt-5.5` xhigh always for critical plans |
| Marketing Director | `openai-codex/gpt-5.5-fast` | `openai-codex/gpt-5.5` for premium offer/positioning |
| Copywriter | `openai-codex/gpt-5.5-fast` | `openai-codex/gpt-5.5` for final customer-facing copy |
| Research / market signal | `xai/grok-4.20-beta-latest-reasoning` | non-reasoning variant for cheap scans only |
| Frontend/UI | `kimi/kimi-code` or `kimi-coding/k2p5` | `openai-codex/gpt-5.5` for product-critical UX decisions |
| Infra | `zai/glm-5.1` | `openai-codex/gpt-5.5` for high-risk deploy/cutover planning |
| Code implementation | `codex-cli/gpt-5.4` | `openai-codex/gpt-5.5` for architecture ambiguity |
| Code review/security | `codex-cli/gpt-5.4` | `openai-codex/gpt-5.5` for release blockers |

## Immediate fixes
1. Add quality gate to the Prague Executive Hackathon pilot:
   - final artifact must pass human-readable formatting review.
   - no raw JSON unless requested.
2. Upgrade `marketing-director` away from mini before serious launch artifacts.
3. Split copywriting from coding models; keep `kimi` for coding/UI, not premium sales copy by default.
4. Run a small benchmark pack:
   - offer page copy
   - outreach sequence
   - ICP note
   - executive one-pager
   Score: clarity, specificity, polish, conversion strength, formatting cleanliness.

## Config-change policy
- Model updates are configuration changes.
- Apply only with explicit founder approval or clear operational directive.
- After model config changes: restart gateway and run `openclaw doctor --non-interactive`.
