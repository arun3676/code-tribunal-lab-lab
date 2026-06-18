# Code Council

**Multi-model code analysis — and Band-powered intent review for AI-generated diffs.**

Code Council is a split-stack monorepo: a Next.js frontend and a FastAPI backend that stream multiple LLM opinions over the same code. The flagship hackathon mode, **Tribunal**, adds a Band-coordinated agent court that checks whether a diff actually matches the original ticket — not just whether the code looks correct.

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Features

| Mode | Status | Description |
|------|--------|-------------|
| **Solo** | Live | Stream one model's analysis with optional static security/performance scan |
| **Council** | Live | Run 2–4 models in parallel; consensus ribbon and agreement matrix |
| **Multimodal** | Live | Paste or drop an image for vision-model analysis |
| **Tribunal** | Planned | Band multi-agent intent-conformance review — see [hackathon plan](docs/hackathon/plan.md) |

## Stack

| Layer | Tech | Deploy |
|-------|------|--------|
| Frontend | Next.js 15, TypeScript, Tailwind, Monaco | Vercel |
| Backend | FastAPI, SSE streaming, Pydantic | Railway |
| Models | Gemini, DeepSeek, Mercury, Kimi | API keys |

## Repository layout

```
apps/api/     FastAPI backend (code_council package)
apps/web/     Next.js frontend
docs/         Architecture, structure, hackathon plans
legacy/       Archived Streamlit-era modules (portfolio)
```

Full tree: [`docs/STRUCTURE.md`](docs/STRUCTURE.md)

## Quick start

```bash
git clone https://github.com/arun3676/code-tribunal-lab-lab.git
cd code-tribunal-lab-lab
cp .env.example .env    # add LLM API keys
docker compose up
```

Open [http://localhost:3000](http://localhost:3000). API health: [http://localhost:8000/health](http://localhost:8000/health).

### Environment variables

Copy [`.env.example`](.env.example) to `.env` at the repo root:

```env
GEMINI_API_KEY=
DEEPSEEK_API_KEY=
MERCURY_API_KEY=
Kimi_API_KEY=
NEXT_PUBLIC_API_URL=http://localhost:8000
ALLOWED_ORIGINS=http://localhost:3000
```

See [`apps/api/.env.example`](apps/api/.env.example) for Railway-only vars and [`DEPLOYMENT.md`](DEPLOYMENT.md) for production.

## Deploy

Two services:

- `apps/api` → Railway (Dockerfile)
- `apps/web` → Vercel (`NEXT_PUBLIC_API_URL` → Railway URL)

Full checklist: [`DEPLOYMENT.md`](DEPLOYMENT.md)

## Documentation

| Doc | Purpose |
|-----|---------|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design and endpoints |
| [STRUCTURE.md](docs/STRUCTURE.md) | Repo layout and conventions |
| [hackathon/goal.md](docs/hackathon/goal.md) | Tribunal hackathon goals |
| [hackathon/plan.md](docs/hackathon/plan.md) | Tribunal build execution plan |

## Legacy

The [`legacy/`](legacy/) directory preserves earlier Streamlit and analyzer experiments. It is not part of the active runtime.

## License

MIT — see [LICENSE](LICENSE).
