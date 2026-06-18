# Target Layout

Current monorepo layout (updated for Tribunal hackathon path). See [`STRUCTURE.md`](./STRUCTURE.md) for the canonical tree.

```text
code-tribunal-lab-lab/
├── apps/
│   ├── web/                 # Next.js 15 frontend (Vercel)
│   └── api/                 # FastAPI backend (Railway)
├── legacy/                  # Archived modules + assets/
├── docs/
│   ├── ARCHITECTURE.md
│   ├── STRUCTURE.md
│   ├── TRIAGE.md
│   ├── MIGRATION_PLAN.md
│   └── hackathon/
│       ├── goal.md
│       └── plan.md
├── .github/workflows/ci.yml
├── docker-compose.yml
├── .env.example
├── LICENSE
├── README.md
└── DEPLOYMENT.md
```

## Backend layout

```text
apps/api/
├── code_council/
│   ├── server.py
│   ├── analyzer.py
│   ├── models.py, prompts.py, utils.py, language.py
│   ├── multimodal.py, fixes.py, github.py
│   ├── scanners/            # security.py, performance.py
│   └── tribunal/            # planned: Band intent-conformance workflow
├── pyproject.toml
├── Dockerfile
├── railway.json
└── README.md
```

## Frontend layout

```text
apps/web/
├── src/
│   ├── app/
│   │   ├── page.tsx         # Solo + Council
│   │   ├── about/
│   │   ├── design/          # Design tokens (dev reference)
│   │   └── tribunal/        # planned: War Room UI
│   ├── components/
│   │   ├── effects/matrix-rain.tsx
│   │   └── shell/app-shell.tsx
│   └── lib/
│       └── api.ts           # JSON + SSE client
├── package.json
└── vercel.json
```

## Migration rules

- `apps/api/` is the only Python runtime.
- `apps/web/` is the only user-facing UI.
- `legacy/` is read-only archive — never imported by apps.
- Root `code_analyzer/` was removed; ported code lives in `apps/api/code_council/`.
- Tribunal code goes under `code_council/tribunal/`, not a sibling package.

## Aspirational (post-Tribunal)

These paths from the original migration plan are optional extractions:

- `src/components/council/` — split from large `page.tsx`
- `src/components/verdict/` — verdict stamp, trust meter, ledger
- `src/lib/sse.ts` — extract SSE parser from `api.ts`

Not required for hackathon submission.
