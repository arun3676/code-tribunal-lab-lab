# Repository structure

Professional layout for the Code Council monorepo. Active runtime code lives only under `apps/`.

```
code-tribunal-lab-lab/
├── apps/
│   ├── api/                    # FastAPI backend → Railway
│   │   ├── code_council/       # Python package (single runtime)
│   │   │   ├── server.py
│   │   │   ├── analyzer.py
│   │   │   ├── scanners/
│   │   │   ├── github.py       # Reserved for Tribunal docket ingestion
│   │   │   └── tribunal/       # Planned: Band multi-agent workflow
│   │   ├── Dockerfile
│   │   ├── pyproject.toml
│   │   └── README.md
│   └── web/                    # Next.js 15 frontend → Vercel
│       ├── src/
│       │   ├── app/            # Routes: /, /about, /design, /tribunal (planned)
│       │   ├── components/     # shell/, effects/
│       │   └── lib/            # api.ts (SSE client)
│       └── README.md
├── docs/
│   ├── ARCHITECTURE.md         # System design
│   ├── STRUCTURE.md            # This file
│   ├── TARGET_LAYOUT.md        # Layout reference
│   ├── TRIAGE.md               # Migration decisions
│   ├── MIGRATION_PLAN.md       # Historical migration spec
│   └── hackathon/
│       ├── goal.md             # Tribunal hackathon goals
│       └── plan.md             # Tribunal execution plan
├── legacy/                     # Archived pre-monorepo code (~127 KB)
│   ├── assets/                 # Design + test images
│   └── code_analyzer/          # Streamlit-era modules
├── .github/workflows/ci.yml
├── docker-compose.yml
├── .env.example
├── LICENSE
├── README.md
└── DEPLOYMENT.md
```

## What belongs where

| Location | Rule |
|----------|------|
| `apps/api/code_council/` | All Python runtime code |
| `apps/web/src/` | All user-facing UI |
| `legacy/` | Read-only archive; never imported by apps |
| `docs/` | Architecture, migration, hackathon planning |
| Root | Only monorepo config, README, compose, license |

## Removed from the repo (intentionally)

- Root `code_analyzer/` — duplicate of ported modules; use `apps/api/code_council/` and `legacy/`
- Virtualenvs (`.venv/`, `testenv/`) — create locally; never commit
- Build output (`.next/`, `*.egg-info/`, `node_modules/`) — gitignored
- Log files — gitignored

## Next build target

**Code Council Tribunal** — Band-powered intent-conformance review for AI-generated code. See [`docs/hackathon/plan.md`](./hackathon/plan.md).
