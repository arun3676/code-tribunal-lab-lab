# Code Council Web

Next.js 15 frontend for [Code Council](https://github.com/arun3676/code-tribunal-lab-lab).

## Routes

| Path | Description |
|------|-------------|
| `/` | Solo + Council analysis (Monaco editor, SSE streaming) |
| `/about` | Project overview |
| `/design` | Internal design tokens / color reference |
| `/tribunal` | *(planned)* Band War Room for intent-conformance review |

## Local run

```bash
cd apps/web
cp .env.example .env.local
pnpm install
pnpm dev
```

Set `NEXT_PUBLIC_API_URL` to your API origin (default `http://localhost:8000`).

Or from repo root: `docker compose up`

## Stack

- Next.js 15 App Router, TypeScript, Tailwind CSS
- Monaco editor, manual SSE parsing (`src/lib/api.ts`)
- Matrix-themed UI (`components/effects/matrix-rain.tsx`)

## Deploy

Vercel — see root [`DEPLOYMENT.md`](../../DEPLOYMENT.md).
