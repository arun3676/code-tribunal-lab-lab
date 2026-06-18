# Code Council API

FastAPI backend for [Code Council](https://github.com/arun3676/code-tribunal-lab-lab).

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/models` | Available LLM providers |
| POST | `/analyze` | Single-model analysis (SSE) |
| POST | `/council` | Multi-model council (SSE) |
| POST | `/scan` | Static security + performance scan |
| POST | `/multimodal` | Image analysis |

Tribunal endpoints (`/tribunal/*`) are planned under `code_council/tribunal/` — see [`docs/hackathon/plan.md`](../../docs/hackathon/plan.md).

## Local run

```bash
cd apps/api
python -m venv .venv
.venv/Scripts/activate   # Windows
pip install -e .
cp .env.example .env     # fill API keys
uvicorn code_council.server:app --reload --port 8000
```

Or from repo root: `docker compose up api`

## Package layout

```
code_council/
├── server.py       # FastAPI app
├── analyzer.py     # Multi-provider LLM orchestration
├── scanners/       # Security + performance static analysis
├── multimodal.py   # Vision analysis
├── github.py       # GitHub fetch helpers (reserved for Tribunal docket)
└── tribunal/       # (planned) Band multi-agent intent review
```

## Deploy

Railway — see root [`DEPLOYMENT.md`](../../DEPLOYMENT.md).
