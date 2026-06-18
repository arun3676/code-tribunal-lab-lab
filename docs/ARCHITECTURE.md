# Code Council Architecture

## Overview

Code Council is a split-stack monorepo:

- `apps/web` — Next.js 15 frontend (Vercel)
- `apps/api` — FastAPI backend (Railway)
- `docs` — architecture and Tribunal hackathon plans

The product is intentionally **stateless**:

- No database
- No vector store
- No server-side history
- Optional `localStorage` persistence only in the frontend

## Frontend Stack

- Next.js 15
- TypeScript
- Tailwind CSS
- shadcn/ui primitives
- Monaco editor
- Streaming UI via manual SSE parsing over `fetch`

The frontend is responsible for:

- Editor UX
- Solo analysis flow
- Council view
- Multimodal image upload flow
- Static scan presentation
- Local-only persistence of the last session state

## Backend Stack

- FastAPI
- Uvicorn
- OpenAI-compatible SDK clients for DeepSeek, Mercury, and Kimi
- Google Generative AI client for Gemini
- `python-dotenv`
- `pydantic`
- `sse-starlette`
- Existing ported analyzer modules

The backend is responsible for:

- Provider availability discovery
- Streaming single-model analysis
- Streaming parallel council analysis
- Multimodal image analysis
- Security and performance scans
- Lightweight request limiting and CORS

## Runtime Boundaries

### Kept Capabilities

- Multi-model code analysis
- Language detection
- Security scanning
- Performance scanning
- Multimodal image analysis
- AI-generated fix suggestions
- Council consensus/disagreement rendering support

### Removed Capabilities

- Streamlit UI
- SQLite dashboards
- ChromaDB/RAG
- Cloud analyzer runtime
- Container analyzer runtime
- Framework analyzer runtime
- CI/CD CLI runtime
- OpenAI and Anthropic provider support
- GitHub endpoint in the public API surface

## Environment Variables

### Backend

- `GEMINI_API_KEY`
- `DEEPSEEK_API_KEY`
- `MERCURY_API_KEY`
- `Kimi_API_KEY`
- `ALLOWED_ORIGINS`
- `PORT`

### Frontend

- `NEXT_PUBLIC_API_URL`

## Deployment Topology

### Railway

The backend runs with:

```bash
uvicorn code_council.server:app --host 0.0.0.0 --port $PORT --workers 1
```

### Vercel

The frontend builds from `apps/web` and calls the Railway backend via `NEXT_PUBLIC_API_URL`.

## Design Constraints

- Budget stays below the project cap by using the cheapest active provider/model per vendor.
- Missing provider keys should disable models rather than crash the backend.
- SSE is the transport for long-running analysis so the frontend can stream tokens live.
- The backend should remain import-clean without `langchain`, `chromadb`, or legacy UI dependencies.

## Repository layout

See [`STRUCTURE.md`](./STRUCTURE.md) for the canonical directory tree and file placement rules.

## Planned: Tribunal mode

**Code Council Tribunal** (Band of Agents Hackathon, Track 2) adds intent-conformance review:

- Package: `apps/api/code_council/tribunal/`
- UI route: `apps/web/src/app/tribunal/`
- Endpoints: `GET /tribunal/fixtures`, `POST /tribunal/run` (SSE)
- Band as coordination layer: @mentions, structured events, mid-trial recruitment

Execution plan: [`docs/hackathon/plan.md`](./hackathon/plan.md)
