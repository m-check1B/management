# CouncilNow Dummy Customer E2E — 2026-04-26

_Status: Green after report JSON safety, model-routing stabilization, and JSON repair deployment_

## Production deployment

Latest production deployment included:

- report JSON/export safety patch
- role completion token budget fix
- first-customer-safe model routing v11
- automatic malformed advisor JSON repair

Production health after deploy:

```bash
curl -sS https://councilnow.com/health
# {"status":"ok","version":"0.4.0","database":"ok",...}
```

## Final dummy-customer run

- Dummy email: `tba-dummy-20260426t061910z@example.com`
- Dummy user ID: `265ed4b8-f09c-48ce-b403-e1a923ff010f`
- Session ID: `d8c7ba2a-59b0-4317-bae7-298dbcfa2d43`
- Share URL: `https://councilnow.com/shared/b531e895-7c21-49d1-b22a-77b89e3503e3`
- Score: `27`
- Confidence: `6.0`
- Public marker hits: `[]`
- Local JSON evidence: `/tmp/councilnow-dummy-session-20260426T061910Z.json`
- Markdown export evidence: `/tmp/councilnow-dummy-session-20260426T061910Z.md`

Run completed successfully:

```json
{
  "ok": true,
  "out": "/tmp/councilnow-dummy-session-20260426T061910Z.json",
  "email": "tba-dummy-20260426t061910z@example.com",
  "user_id": "265ed4b8-f09c-48ce-b403-e1a923ff010f",
  "session_id": "d8c7ba2a-59b0-4317-bae7-298dbcfa2d43",
  "share_url": "https://councilnow.com/shared/b531e895-7c21-49d1-b22a-77b89e3503e3",
  "score": 27,
  "confidence": 6.0,
  "public_marker_hits": []
}
```

## Report formatting verification

Markdown export check:

```json
{
  "input": "/tmp/councilnow-dummy-session-20260426T061910Z.json",
  "output": "/tmp/councilnow-dummy-session-20260426T061910Z.md",
  "bytes": 48350,
  "marker_hits": []
}
```

Public shared page check:

```bash
curl -sSL https://councilnow.com/shared/b531e895-7c21-49d1-b22a-77b89e3503e3
# HTTP/2 200
# no hits for: ```json, ```, "option_a", "updated_prediction", raw_response, parse_error, metadata envelope, schema commentary
```

## Code/test evidence

Commits in `/Users/matejhavlin/github/applications/advisory-council-app`:

- `8bfa014 Fix report formatting and upgrade council models`
- `034e784 Prevent raw advisor JSON in reports`
- `fa82a57 Respect role completion token budgets`
- `0802de7 Stabilize CouncilNow first-customer model routing`
- `1aa463d Repair malformed advisor JSON drafts`

Tests run:

```bash
cd backend
.venv/bin/python -m pytest tests/test_council_model_config.py tests/test_council_runner_json.py tests/test_report_verifier.py tests/test_sessions_api.py -q
# 59 passed

cd frontend
node --test --experimental-strip-types tests/session-report-formatting.test.mjs
# 3 passed
```

## Model routing state

`council-models.json` is now version `11`.

Paid/customer-facing routing:

- `strong`
  - strategist/contrarian: `x-ai/grok-4.20`
  - pragmatist/customer/operator: `openai/gpt-5.4-mini`
  - analyst/synthesis: `openai/gpt-5.5`
  - verifier: `x-ai/grok-4.20`
  - JSON repair: `openai/gpt-5.4-mini`
- `frontier`
  - analyst/synthesis: `openai/gpt-5.5-pro`
  - pragmatist/customer/operator: `openai/gpt-5.4`
  - verifier: `x-ai/grok-4.20`
  - JSON repair: `openai/gpt-5.4-mini`

Reasoning: the earlier version used GLM/Gemini/MiMo lanes for some roles; live dummy tests exposed malformed/empty structured outputs and truncation risk. Routing was adjusted to keep Grok/GPT-5.5 for high-leverage reasoning while using GPT-5.4/GPT-5.4 Mini for stable structured advisor JSON.

## Answer to the formatting question

Yes — the JSON formatting issue is fixed in the report/export layer and protected with regression tests. The initial synthesis-level problem was fixed first; then the deeper full-report leak from advisor `raw_response` / `parse_error` fallback was found, fixed, deployed, and verified against a live dummy customer.

## Remaining go/no-go items before external outreach

- Product/dummy-customer path: green.
- Report formatting: green.
- CouncilNow sender domain: DNS added, Resend status still pending at first check; follow-up cron scheduled.
- Founder mail Proton cutover: blocked on Proton UI-generated TXT/DKIM values.
