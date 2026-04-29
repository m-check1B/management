# AgentJack Production Deploy Recipe — agentjack-control

_Date: 2026-04-29 09:54 Europe/Prague_
_Status: recipe produced; build not started_

## Services

| Service | Stack | Port | Health |
|---|---|---|---|
| AG WebUI | Node.js (SvelteKit static build) | 4273 | `GET /health` |
| AgentJack Backend | Python aiohttp + Hermes gateway | 8801 | `GET /health`, `GET /status` |

## Directory mapping (dev → prod)

| Dev (Mac) | Prod (agentjack-control) | Purpose |
|---|---|---|
| `~/github/agentjack` | `/srv/kraliki/agentjack/source` | Repo checkout |
| `~/.agentjack/state` | `/var/lib/kraliki/agentjack/home` | Runtime state |
| `~/.agentjack/.env` | `/etc/kraliki/agentjack/agentjack.env` | AgentJack env |
| `~/.hermes/profiles/axis/.env` | `/etc/kraliki/agentjack/hermes-axis.env` | Hermes profile env |
| `~/github/secrets/kiki.env` | `/etc/kraliki/agentjack/secrets.env` | API keys |
| `~/.agentjack/live-models.env` | `/etc/kraliki/agentjack/live-models.env` | Model overrides |
| `axis-workspace` | `/var/lib/kraliki/agentjack/workspaces` | Workspace dir |
| `ag-webui/build` | `/srv/kraliki/agentjack/source/ag-webui/build` | Built frontend |

## Env key names (no values)

**WebUI:** `HOST`, `PORT`, `PUBLIC_URL`, `ORIGIN`, `AGENTJACK_ROOT`, `AGENTJACK_PYTHON_BIN`, `AGENTJACK_RUNTIME_MODEL`, `AGENTJACK_VISION_MODEL`, `AGENTJACK_WORKSPACE`, `AGENTJACK_ALLOW_EXTERNAL_READS`, `AGENTJACK_ALLOW_PROTECTED_READS`, `AGENTJACK_DATA_DIR`, `AG_WEBUI_ALLOW_ENV_AUTH`, `AG_WEBUI_EMAIL`, `AG_WEBUI_NAME`

**API keys (from secrets):** `GROQ_API_KEY`, `GEMINI_API_KEY`, `KIMI_API_KEY`, `KIMI_BASE_URL`, `OPENROUTER_API_KEY`, `XIAOMI_MIMO_API_KEY`, `XIAOMI_MIMO_BASE_URL`, `XAI_API_KEY`

**Backend:** `AGENTJACK_STATE_DIR`, `AGENTJACK_TELEGRAM_BOT_TOKEN`

## Build steps

```bash
cd /srv/kraliki/agentjack/source

# Backend
python3 -m venv .venv
.venv/bin/pip install -e .

# Frontend
cd ag-webui && npm install && npm run build && cd ..
```

## Systemd (template)

- `agentjack-webui.service` — ExecStart runs `scripts/agentjack-ag-webui-server.sh` adapted for Linux paths, EnvironmentFile loads `/etc/kraliki/agentjack/*.env`
- `agentjack-runtime.service` — ExecStart runs `uv run python -m agentjack runtime-start --path .../openclaw.json --profile axis`

## Verification gates

```bash
curl -fsS http://127.0.0.1:4273/health   # WebUI
curl -fsS http://127.0.0.1:8801/health   # Backend
```

## Blockers to resolve before build

1. **`inference-plane` local dep** — `pyproject.toml` has `inference-plane @ file:///Users/matejhavlin/...`. Won't resolve on prod. Remove or publish it.
2. **Hermes CLI** — Referenced but unclear if standalone binary, Python package, or needs separate install.
3. **`scripts/agentjack-ag-webui-server.sh`** — Hardcodes macOS paths (`/opt/homebrew/bin`, `$HOME`). Needs Linux adaptation.
4. **Reverse proxy** — Need nginx/caddy/Traefik in front of 4273 + 8801 with TLS.
5. **Zitadel auth** — WebUI has Zitadel auth defaults; need to point at internal Zitadel clone.
6. **`openclaw.json`** — Config file contents/schema unknown; may need prod-specific values.
7. **Dedicated system user** — Decide: `agentjack` system user or root.
8. **DNS/TLS** — `agentjack.kraliki.com` provisioning after checkpoint.
