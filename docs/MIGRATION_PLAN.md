# Code Council — Migration & Modernization Plan

> Portfolio refresh of the LLM Code Analyzer. Reframes the product, migrates Streamlit → Next.js + FastAPI, and rebuilds the Matrix UI as a refined product surface. Each phase below is a self-contained prompt designed to be pasted into Cursor (or any agentic coding setup) one at a time.

---

## Strategic Direction

**Old framing:** "LLM Code Analyzer" — a Streamlit app that runs code through multiple LLMs and shows analysis. Looks like a junior version of CodeRabbit/Greptile.

**New framing:** **Code Council** — a destination tool where engineers watch frontier models reason about the same code and see where they agree vs. disagree. Not a PR bot. Not an autofixer. A sandbox for exploring how different AIs think about your code.

**Why this is different from PR-bot startups:**

| Code review startups (Greptile, CodeRabbit, Cursor Bugbot) | Code Council |
|---|---|
| One AI verdict, optimized for PR-time | Multi-model panel, optimized for exploration |
| Lives inline in GitHub / IDE | Destination tool you visit on purpose |
| Goal: ship fixes fast | Goal: see disagreement, learn from it |
| Workflow: automated, in the background | Workflow: hands-on, you sit with it |

**Budget constraint:** < $0.90 across all API keys. Use the cheapest model per provider. The app must work end-to-end; accuracy is secondary.

**Architecture:**
- **Frontend** (Vercel free): Next.js 15 App Router, TypeScript, Tailwind, shadcn/ui, Monaco editor
- **Backend** (Railway $5): FastAPI, existing analyzer modules, SSE streaming
- **No DB**. Stateless. Optional `localStorage` history on the client.
- **No vector store**. ChromaDB cut.

**Visual direction:** Refined Matrix. Phosphor green (`#00FF66`-ish, slightly cooler than the old `#00ff41`), JetBrains Mono everywhere, subtle matrix rain (5–8% opacity, slower), tight spacing (`text-sm`, `gap-3`), real shadcn primitives underneath. Think terminal meets Linear, not 1999 hacker GIF.

**Active LLM Providers (4 working, tested 2026-05-01):**

| Provider | Model | Pricing | Notes |
|----------|-------|---------|-------|
| Gemini | `gemini-2.5-flash` | Free tier / pennies | Vision-capable |
| DeepSeek | `deepseek-chat` (V3) | $0.27/M in, $1.10/M out | Cheapest text model |
| Mercury | `mercury-coder-small` | Free (beta) | Code-specialized, fastest |
| Kimi | `moonshot-v1-8k` | Free tier (¥0.006/1K tokens) | OpenAI-compatible, vision models available |

**Removed providers:** OpenAI (quota exhausted), Anthropic (invalid key)

**Features kept from original:**
- Multi-LLM code analysis (DeepSeek, Mercury, Gemini, Kimi — 4 models)
- Language detection
- Regex + AST security scanning
- Pattern-based performance scanning
- Multimodal vision analysis (Gemini 2.5 Flash + Kimi vision models)
- AI-powered fix suggestions with diffs

**Features cut:**
- SQLite dashboard / quality_metrics persistence
- CI/CD CLI integration
- Container analyzer (Dockerfile, K8s YAML)
- Cloud analyzer (AWS/Azure/GCP)
- ChromaDB RAG (ephemeral, never paid for itself)
- Framework analyzer (small surface, low signal)
- GitHub file/repo fetch (GitHub token expired, no API access)
- OpenAI / Anthropic model support (keys dead)

**Features added (the differentiators):**
- **Council View** — side-by-side model verdicts with consensus/disagreement highlighting
- **Streaming responses** — tokens flow in live as each model responds
- **Confidence rendering** — quality scores as phosphor meters, not numbers
- **Refined Matrix theme** — smaller density, real product polish

---

## Phase 0 — Repository Audit & New Layout Decision

**Goal:** Produce a written triage of every file in the current repo and a target directory layout for the split-stack rebuild.

**Why it matters:** Without this, Phase 1 turns into a free-for-all and the rebuild drifts.

**Prompt:**
```
You have access to the existing llm-code-analyzer Python repo (Streamlit + monolith).
Read every file in /code_analyzer and produce two artifacts:

ARTIFACT 1 — /docs/TRIAGE.md
A table with columns: File | Decision | Reason | Migration Note

Decisions are exactly one of:
  PORT     — code moves to the new FastAPI backend largely as-is
  REWRITE  — concept survives but the implementation is replaced
  ARCHIVE  — code stays in /legacy/ as portfolio evidence but is not wired up
  DELETE   — genuinely dead code (auto-generated, broken stubs, true duplicates)

Apply these rules:
  PORT     = main.py, models.py, prompts.py, utils.py, security_analyzer.py,
             performance_analyzer.py, multimodal_analyzer.py, language_detector.py,
             github_analyzer.py, fix_suggestions.py
  REWRITE  = web/app.py (Streamlit → discarded, replaced by Next.js frontend),
             advanced_analyzer.py (refactored as a thin FastAPI service layer)
  ARCHIVE  = dashboard.py, ci_cd_integration.py, framework_analyzer.py,
             cloud_analyzer.py, container_analyzer.py, evaluator.py,
             web/templates/dashboard.html
  DELETE   = openai_fix.py (replace with proper httpx config), chroma_db/

ARTIFACT 2 — /docs/TARGET_LAYOUT.md
The target monorepo layout, exactly:

  code-council/
  ├── apps/
  │   ├── web/                 # Next.js 15 frontend (Vercel)
  │   └── api/                 # FastAPI backend (Railway)
  ├── legacy/                  # ARCHIVE bucket from triage
  ├── docs/
  │   ├── TRIAGE.md
  │   ├── TARGET_LAYOUT.md
  │   └── ARCHITECTURE.md
  ├── .github/workflows/       # CI for type-check + lint
  ├── README.md
  └── .gitignore

Do NOT move any files yet. Phase 0 is decisions only.
```

**Acceptance:** Two markdown files in `/docs/` you'd be willing to defend in a code review.

---

## Phase 1 — Monorepo Skeleton & Backend Port

**Goal:** Stand up the new monorepo, move PORT-decision Python modules into `apps/api/`, archive the rest, and verify everything still imports.

**Why it matters:** This is the only phase that touches Python heavily. After this, all subsequent work is FastAPI wiring + Next.js, which is much faster.

**Prompt:**
```
Create the monorepo layout from /docs/TARGET_LAYOUT.md and migrate the existing
Python code per /docs/TRIAGE.md.

Steps:

1. Create the directory structure (apps/web/, apps/api/, legacy/, docs/, .github/).

2. Move PORT files into apps/api/code_council/:
     main.py → apps/api/code_council/analyzer.py    (rename CodeAnalyzer → Analyzer)
     models.py → apps/api/code_council/models.py
     prompts.py → apps/api/code_council/prompts.py
     utils.py → apps/api/code_council/utils.py
     security_analyzer.py → apps/api/code_council/scanners/security.py
     performance_analyzer.py → apps/api/code_council/scanners/performance.py
     multimodal_analyzer.py → apps/api/code_council/multimodal.py
     language_detector.py → apps/api/code_council/language.py
     github_analyzer.py → apps/api/code_council/github.py
     fix_suggestions.py → apps/api/code_council/fixes.py

3. Move ARCHIVE files into /legacy/ unchanged. Add /legacy/README.md explaining
   that this code represents earlier iterations (dashboard, CI/CD, container/cloud
   analyzers, framework analyzer) preserved as engineering evidence but not part
   of the current runtime.

4. DELETE: chroma_db/, openai_fix.py (will be replaced inline in analyzer.py with
   httpx-configured clients), the old web/ directory entirely, the old Dockerfile
   (will be rewritten in Phase 6), render.yaml.

5. Create apps/api/pyproject.toml with the slim dep set:
     fastapi, uvicorn[standard], httpx, openai, google-generativeai,
     python-dotenv, pillow, requests, sse-starlette, pydantic
   NOTE: `openai` is still needed — DeepSeek and Mercury both use the
   OpenAI-compatible SDK. `anthropic` is CUT (key is dead).
   Cut: streamlit, langchain*, chromadb, sentence-transformers, torch,
   transformers, scikit-learn, scipy, matplotlib, gitpython, plotly, pandas,
   tiktoken, anthropic. (langchain templates are tiny; reimplement the two
   prompt templates inline as f-strings in prompts.py.)

6. Update all import paths. Run `python -c "import code_council"` from apps/api/
   and verify zero import errors. Fix any langchain references by inlining the
   PromptTemplate calls.

7. Write /docs/ARCHITECTURE.md describing: monorepo layout, frontend stack,
   backend stack, deployment targets (Vercel + Railway), env var topology, the
   no-DB / stateless decision.

8. Add a root .gitignore covering Python (__pycache__, .venv, .env), Node
   (node_modules, .next, .turbo), and editor cruft.

Do NOT write FastAPI routes yet. Do NOT touch apps/web/ yet. End state: imports
clean, legacy quarantined, docs in place.
```

**Acceptance:** `cd apps/api && python -c "from code_council.analyzer import Analyzer"` runs with no errors and no langchain/chromadb imports.

---

## Phase 2 — FastAPI Service Layer

**Goal:** Wrap the ported analyzer modules in a clean FastAPI app with streaming endpoints and proper CORS.

**Why it matters:** This is the contract the Next.js frontend will consume. Get it right once, never touch it again.

**Prompt:**
```
Build the FastAPI service in apps/api/. The frontend (Vercel, https://code-council.vercel.app
in production, http://localhost:3000 in dev) will be the only client.

Create apps/api/code_council/server.py with these endpoints:

GET  /health
     → { status: "ok", version: <git sha or "dev"> }

GET  /models
     → list of available model identifiers based on which env vars are set, e.g.
       [
         { "id": "gemini-2.5-flash", "provider": "gemini", "available": true,
           "display": "Gemini 2.5 Flash", "color": "#4285F4" },
         { "id": "deepseek-chat", "provider": "deepseek", "available": true,
           "display": "DeepSeek V3", "color": "#00FF66" },
         { "id": "mercury-coder-small", "provider": "mercury", "available": true,
           "display": "Mercury Small", "color": "#FF8A65" },
         { "id": "moonshot-v1-8k", "provider": "kimi", "available": true,
           "display": "Kimi v1-8k", "color": "#9C27B0" }
       ]
     A model is "available" iff its API key env var is present. The frontend
     uses this to disable unselectable models.
     NOTE: 4 providers active. No OpenAI or Anthropic.

POST /analyze
     Body: { code: string, language?: string, model: string, mode: "quick"|"thorough" }
     → SSE stream of events:
        event: token   data: { delta: "..." }
        event: parsed  data: { quality_score, bugs, suggestions, documentation }
        event: fixes   data: [{ ... fix suggestion ... }]   (thorough only)
        event: done    data: { duration_ms }
     Implementation: stream raw tokens to the client as they arrive from the
     provider SDK; once the stream completes, parse the accumulated text via
     utils.parse_llm_response and emit the parsed event. This gives the UI
     both the live-typing effect AND the structured data.

POST /council
     Body: { code: string, language?: string, models: string[], mode: "quick"|"thorough" }
     → SSE stream that fans out to all selected models in parallel, multiplexing
       events with a `model` field on every payload:
        event: token   data: { model: "deepseek-chat", delta: "..." }
        event: parsed  data: { model: "deepseek-chat", quality_score, ... }
        event: done    data: { model: "deepseek-chat", duration_ms }
        event: all_done data: { total_duration_ms }
     Use asyncio.gather under the hood. Each model's stream is independent;
     a slow model does not block faster ones.

POST /multimodal
     multipart/form-data: image file + optional prompt
     → JSON: { analysis, code_extracted, suggestions, model }
     NOTE: Only Gemini 2.5 Flash supports vision. No model selection needed.

(REMOVED: POST /github — GitHub token expired, endpoint cut from scope)

POST /scan
     Body: { code: string, language?: string }
     → Synchronous JSON (no streaming, this is regex + AST, completes in <100ms):
       {
         security: { vulnerabilities: [...], risk_score: 0-100 },
         performance: { issues: [...], overall_score: 0-100 }
       }

CORS: allow http://localhost:3000 + the production Vercel URL via env var
ALLOWED_ORIGINS. Methods: GET, POST. Headers: Content-Type, Authorization.

Configuration:
  - Read API keys from environment: GEMINI_API_KEY, DEEPSEEK_API_KEY,
    MERCURY_API_KEY, Kimi_API_KEY (4 active providers).
  - On startup, log which providers are available (don't crash on missing keys —
    just disable that model in /models).
  - Add a simple in-memory rate limiter: 30 requests/min per IP. Reject with 429.
  - Budget note: use cheapest models only (gemini-2.5-flash, deepseek-chat,
    mercury-coder-small, moonshot-v1-8k). Total budget < $0.90.

Run command for Railway:
  uvicorn code_council.server:app --host 0.0.0.0 --port $PORT --workers 1

Add apps/api/Dockerfile (Python 3.11-slim, no system deps beyond ca-certificates,
COPY pyproject.toml first for layer caching, pip install, COPY rest, CMD uvicorn).

Add apps/api/.env.example listing every var with a one-line comment.

Test locally:
  curl http://localhost:8000/health
  curl http://localhost:8000/models
  curl -N -X POST http://localhost:8000/analyze \
       -H "Content-Type: application/json" \
       -d '{"code":"def add(a,b):return a+b","model":"deepseek-chat","mode":"quick"}'
  (the -N is critical — it disables curl's buffering so you see the stream)
```

**Acceptance:** All four endpoints (/health, /models, /analyze, /council, /multimodal, /scan) return correctly via curl. SSE actually streams (you see tokens arrive over time, not in one chunk). `/models` correctly reflects which API keys are set.

---

## Phase 3 — Next.js Frontend Scaffold + Refined Matrix Theme

**Goal:** Stand up the Next.js app with the visual system. No real features yet — just the chrome.

**Why it matters:** Locking the theme tokens early means every subsequent feature inherits the look automatically. Doing this last leads to inconsistent components.

**Prompt:**
```
Create the Next.js 15 app in apps/web/. App Router, TypeScript strict, Tailwind,
shadcn/ui, Geist + JetBrains Mono fonts.

1. Scaffold:
     pnpm create next-app@latest apps/web --typescript --tailwind --app --src-dir
     cd apps/web && pnpm dlx shadcn@latest init
     Install primitives: button, card, dialog, input, label, select, separator,
     skeleton, tabs, toast, tooltip, dropdown-menu, scroll-area, badge.

2. Theme tokens — apps/web/src/app/globals.css. Define CSS vars on :root:
     --bg:           #050805           (near-black, slight green tint)
     --bg-elevated:  #0A0F0A
     --bg-overlay:   #0F140F
     --border:       #1A2A1A
     --border-hot:   #2A4A2A
     --fg:           #B8FFC8           (soft phosphor for body text — NOT pure green)
     --fg-muted:     #6B8B73
     --fg-dim:       #3D5942
     --accent:       #00FF66           (the hot phosphor, used SPARINGLY — buttons,
                                        active states, focus rings, the matrix rain)
     --accent-soft:  #00FF6622
     --danger:       #FF5577
     --warning:      #FFAA44
     --info:         #66CCFF

   Why two greens: pure #00FF66 everywhere is what makes "matrix UI" look amateur.
   Body text in soft phosphor (#B8FFC8) reads cleanly; saturated green is reserved
   for accents. This is the single most important design decision in this phase.

3. Typography:
     - Headings: Geist Sans, tight tracking (-0.02em)
     - Body: Geist Sans, 14px (text-sm) is the default — the user explicitly
       wants smaller density than the old UI
     - Code, monospace UI elements (model IDs, scores, timestamps): JetBrains Mono

4. Matrix rain — apps/web/src/components/effects/matrix-rain.tsx
     Canvas-based, fixed position behind everything (z-index: -1).
     Constraints (CRITICAL — this is what makes it refined vs. kitsch):
       - Opacity 0.06 max
       - Drop speed: 1 char every 80–120ms (vs. typical 30ms)
       - Character set: katakana + 0-9 (smaller set, less visual noise)
       - Font: 12px JetBrains Mono
       - Color: var(--accent) but rendered with globalAlpha 0.4
       - Pause when document.hidden (battery)
       - Disable entirely on prefers-reduced-motion
     Mount in apps/web/src/app/layout.tsx, wrapped in Suspense.

5. App shell — apps/web/src/app/layout.tsx + a top bar component:
     - Top bar height: 48px (compact, not 64px)
     - Left: monospace wordmark "CODE_COUNCIL" with a blinking caret
     - Center: nothing (resist the urge to add nav — single-page app)
     - Right: model availability dots — one tiny phosphor dot per available
       provider, grayed out when key is missing. Tooltip on hover shows model name.
     - Bottom of viewport: a one-line status bar (12px, monospace) showing:
       "ready" / "analyzing... 2.3s" / "council: 3/5 responding" — terminal vibe.

6. Set up the API client — apps/web/src/lib/api.ts
     - NEXT_PUBLIC_API_URL env var
     - Typed wrappers: getModels(), analyze(), council(), multimodal(),
       githubAnalyze(), scan()
     - For SSE endpoints: return an AsyncIterable<Event> using fetch + ReadableStream
       (NOT EventSource — EventSource doesn't support POST bodies). Parse SSE
       frames manually. This is ~40 lines of code, write it once, reuse everywhere.

7. Add a /design route at apps/web/src/app/design/page.tsx — a kitchen-sink page
   showing every shadcn primitive themed in the new tokens, plus the typography
   scale, plus the color tokens swatched. Helpful for spotting drift later.

8. Verify:
     pnpm dev
     - Matrix rain runs at the configured opacity, doesn't dominate
     - /design route shows all primitives in phosphor theme
     - Top bar wordmark renders, model dots respond to /models endpoint
     - Status bar present
     - prefers-reduced-motion disables the rain

Do NOT build any analysis features yet. This phase is the visual system only.
```

**Acceptance:** Pull up `localhost:3000` and it looks like a finished product that happens to do nothing. `/design` route renders cleanly. Matrix rain is present but not overwhelming.

---

## Phase 4 — Single-Model Analysis Flow

**Goal:** The main page. Paste code, pick a model, watch streamed analysis arrive.

**Prompt:**
```
Build the single-model analysis flow on apps/web/src/app/page.tsx.

Layout (split horizontally on desktop, stacked on mobile):

  LEFT PANE (60% width on ≥lg):
    - Monaco editor, 100% height, dark theme, language auto-detected from a
      tiny dropdown above the editor (default: auto). Use @monaco-editor/react.
      Configure: minimap off, line numbers on, font JetBrains Mono 13px,
      bracket pair colorization on, fontLigatures on.
    - Above the editor: a thin toolbar — language dropdown, mode toggle
      (quick/thorough), char count.
    - Below the editor: the analyze action row (see right pane).

  RIGHT PANE (40%):
    - Header: "VERDICT" in monospace + the selected model's name + a
      latency readout that ticks up while streaming.
    - Body, four sections rendered as the events arrive:
        1. SCORE — a horizontal phosphor meter, 0-100. Animates from 0 to the
           final score over 600ms when the parsed event arrives. Below the
           bar: the numeric score in JetBrains Mono 24px.
        2. BUGS — list of detected bugs, each prefixed with a red bullet.
           Empty state: "no bugs detected" in --fg-dim.
        3. SUGGESTIONS — list of suggestions, each prefixed with a phosphor
           bullet. Empty state similar.
        4. RAW STREAM — collapsed by default, expands to show the raw token
           stream as it came in. Useful for debugging and looks legit.
    - In thorough mode: a fifth section FIXES appears, rendering each fix as
      a card with a unified diff (use shiki or a simple side-by-side render).

Action row (below the editor):
    - Model selector (a dropdown of available models from /models, with the
      provider color rendered as a tiny dot next to each name)
    - Mode toggle (quick | thorough) — quick is default
    - Big "ANALYZE" button — full-width on mobile, fixed-width on desktop.
      Disabled when code is empty. Replaced by "STOP" while streaming
      (AbortController on the fetch).

Behavior:
    - When ANALYZE clicked: hit POST /analyze with SSE. As tokens arrive, append
      them to the raw stream section AND show a typing indicator next to the
      model name in the verdict header.
    - When parsed event arrives: animate the score meter, populate bugs/suggestions.
    - When fixes event arrives (thorough): populate the FIXES section.
    - When done event arrives: stop the latency ticker, show "complete in 2.3s".
    - On error: surface a toast, keep the partial output visible.

Persistence:
    - Save the last code + language + model + mode to localStorage on every
      change. Restore on mount. This is the only state that persists; we have
      no backend DB and don't need one.

Keyboard:
    - Cmd/Ctrl + Enter triggers analyze
    - Cmd/Ctrl + K opens a quick model switcher (cmd-palette style)

Empty state (no code yet):
    - Editor shows a placeholder comment with three example snippets the user
      can click to load: a Python function, a JavaScript async handler, a SQL
      query. Each loads with the appropriate language preset.
```

**Acceptance:** Paste code, click analyze, watch tokens stream in, see the score animate, see bugs and suggestions render. Stop button works. Cmd+Enter works. Reload the page — your last code is still there.

---

## Phase 5 — The Council View (the Differentiator)

**Goal:** The feature that makes this not-a-CodeRabbit-clone. Run all selected models in parallel, render their verdicts side by side, surface where they agree and disagree.

**Why it matters:** This is the only feature in the entire product that competitor tools do not have. It is the thing that will make a portfolio reviewer pause for ten seconds.

**Prompt:**
```
Build the Council View. Add a tab switcher at the top of page.tsx: "SOLO" (the
existing single-model flow) and "COUNCIL". Default to SOLO.

Council layout:

  LEFT PANE (50%):
    - Same Monaco editor as Solo
    - Above editor: "Models in Council" — a multi-select chip group of
      available models. Each chip shows the provider dot + model name. Default
      selection: all available (4 models: Gemini 2.5 Flash, DeepSeek V3,
      Mercury Small, Kimi v1-8k). Capped at 4.
    - Below editor: ANALYZE COUNCIL button. Shows live count while streaming:
      "RUNNING — 2/4 complete" with a progress bar tinted in --accent.

  RIGHT PANE (50%):
    - Grid of N columns where N = number of selected models. Each column is
      a "verdict card" with:
         - Header: provider dot + model name + latency
         - Score meter (animated, smaller than Solo: 16px height vs 24px)
         - Bugs count + Suggestions count as tiny badges
         - A "v" expander button that slides the full bug/suggestion lists down
    - At the top of the grid (above the columns), a CONSENSUS RIBBON:
         A horizontal bar showing the spread of quality scores across models.
         If all scores within ±5: render in --accent ("CONSENSUS").
         If spread is 5–20: render in --warning ("MIXED").
         If spread is >20: render in --danger ("DIVIDED").
         The numerical spread is shown to the right.
    - Below the columns, when all models complete, render an AGREEMENT MATRIX:
         Take every bug+suggestion from every model. Cluster duplicates by
         simple semantic similarity (start with: lowercase + strip punctuation
         + Jaccard similarity on word tokens, threshold 0.55 — good enough for
         the portfolio version, no embedding model needed).
         Render as a table:
            Issue (paraphrased from the longest variant)  | Gemini | DeepSeek | Mercury | Kimi
                                                           ✓        ✗          ✓         ✓
         ✓ = at least one model raised this issue. ✗ = silent.
         Sort rows by "agreement count" descending — most-agreed-upon issues at top.

Behavior:
    - POST /council with the selected models, consume the multiplexed SSE.
    - Each event has a `model` field; route it to the correct column.
    - Columns can finish in any order; render as they complete.
    - The CONSENSUS RIBBON updates after each model's parsed event.
    - The AGREEMENT MATRIX renders only after the all_done event.

Visual details:
    - Each column gets a 1px left border in the model's provider color, full
      height of the card. This is what makes the grid scan as "different
      models" at a glance without needing big colored headers.
    - Use the same score-meter component from Solo, just smaller.
    - The agreement matrix table uses --bg-elevated rows alternating with --bg.
      Checkmarks are --accent, X's are --fg-dim. Text-sm throughout.

Edge cases:
    - If only 1 model is available: hide the COUNCIL tab entirely. The feature
      requires N≥2 to be meaningful.
    - If a model fails mid-stream: render its column with an error state but
      don't halt the others. The matrix still renders, with that model's
      column rendered as "—" instead of ✓/✗.
    - The agreement clustering can over-merge or under-merge. That's fine for
      the portfolio version. Add a small disclaimer in --fg-dim below the
      matrix: "Issues clustered by lexical similarity — interpretation may vary."

Keyboard:
    - Cmd/Ctrl + Enter triggers council analyze (same key as solo)
    - Tab key cycles through columns to expand them
```

**Acceptance:** Select all 4 models (Gemini, DeepSeek, Mercury, Kimi), click ANALYZE COUNCIL, watch all four columns populate at their own pace, see consensus ribbon respond, see agreement matrix render at the end. Visually scannable. The differentiator from PR-bot tools is immediate to anyone looking at it.

---

## Phase 6 — Multimodal Drop & Static Scans

**Goal:** Complete the feature surface. Add the two remaining smaller flows.
(GitHub Quick-Look removed — GitHub token expired, no API access.)

**Prompt:**
```
Add two more entry points to the home page. Place them as cards below the
SOLO/COUNCIL tab switcher in a row of two (stacked on mobile).

Card 1 — DROP IMAGE
    Click target opens a file picker, but also accepts paste-from-clipboard
    anywhere on the page (window-level paste listener checking for image data).
    On image:
      - Show the image preview in a side-by-side: image left, results right
      - POST /multimodal with model selector (2 vision-capable models:
        Gemini 2.5 Flash or Kimi moonshot-v1-8k-vision-preview)
      - Render: extracted code blocks in a Monaco read-only viewer + suggestions list
      - Below extracted code: an "ANALYZE THIS" button that loads the extracted
        code into the main Solo editor and switches back to the Solo tab

Card 2 — STATIC SCAN
    A toggle ribbon that appears above the verdict pane in Solo mode, off by default.
    When toggled on:
      - Calls POST /scan in parallel with /analyze
      - Renders security findings (red dot per finding, severity-colored)
        and performance findings (amber dot per finding) inline in a collapsed
        section above the BUGS list in the verdict
    Scan completes in <100ms (it's regex + AST), so it's a pure visual addition,
    not its own loading state.

Both flows reuse the existing verdict components — no new presentational
work. Just data plumbing.
```

**Acceptance:** Paste an image of code from your clipboard → it analyzes via Gemini or Kimi vision. Toggle static scan → security/perf findings appear above the LLM bugs in real time.

---

## Phase 7 — Polish, Empty States, Micro-interactions

**Goal:** The 10% that takes the product from "demo" to "feels real."

**Prompt:**
```
Apply a polish pass across the entire app. No new features. Every change below
is a micro-interaction or a state most demos forget.

1. Loading states:
   - Replace any Tailwind animate-pulse skeletons with a custom matrix-flicker
     skeleton: monospace block that cycles through random katakana + spaces at
     200ms, dimmed to --fg-dim. Compose it as a single component used everywhere.
   - Score meters animate in with a brief "scrubbing" effect (ticks rapidly
     through random values for 200ms before settling on the real score).

2. Empty states (every panel needs one):
   - Verdict pane before any analysis: ASCII-art-style block reading "AWAITING
     INPUT" in --fg-dim, monospace, centered.
   - Council tab with <2 models selected: "COUNCIL REQUIRES TWO OR MORE
     PARTICIPANTS" + a hint linking to model docs.
   - Static scan disabled: subtle toggle hint, no panel space wasted.

3. Error states:
   - API offline: top bar status flips to red "BACKEND_OFFLINE", every action
     button disables. A retry mechanism polls /health every 5s.
   - Per-model failure in council: that column shows a glitched-out "MODEL
     UNRESPONSIVE" state but the rest of the council continues.

4. Easter eggs (subtle, not embarrassing):
   - Konami code on the home page triggers a 3-second matrix rain density spike
     (opacity 0.06 → 0.18, then back).
   - Type "wake up" in the editor (anywhere, any language) — the caret goes red
     for one second. (No, it doesn't say "follow the white rabbit." Restraint.)

5. Keyboard shortcuts overlay:
   - Press "?" from anywhere → modal listing every shortcut. Style it like a
     terminal man page.

6. Copy affordances:
   - Every code block (in fixes, in extracted code, in raw stream) gets a small
     copy button that appears on hover, monospace, --fg-muted, hover --accent.

7. Toasts:
   - Replace shadcn's default toast styling with monospace, --bg-elevated bg,
     --border-hot border, top-right positioning, 4s auto-dismiss.

8. Favicon + OG image:
   - Favicon: a single phosphor "_" (underscore) on near-black. Generate as an
     SVG, register all the standard sizes.
   - OG image: 1200x630 PNG generated server-side via @vercel/og. Layout:
     wordmark CODE_COUNCIL top-left, tagline "see how the models think"
     bottom-left, faint matrix rain background. Keep it austere.

9. Accessibility:
   - Focus rings: 2px --accent ring on all interactive elements.
   - prefers-reduced-motion: kills matrix rain, score animations, scrubbing.
     Components still work, just statically.
   - Color contrast: every text/bg pair must meet WCAG AA (4.5:1). The
     --fg-muted on --bg combo needs to be checked specifically; bump
     --fg-muted brightness if it fails.

10. Performance:
    - Lazy-load Monaco (it's heavy). Show a code-shaped skeleton while it boots.
    - Lazy-load the matrix rain canvas (defer until after first paint).
    - Bundle audit: run `next build --analyze`, prove the home route is under
      300KB gzipped excluding Monaco.
```

**Acceptance:** Every interaction has a state for empty / loading / error / success. The product feels intentional. Lighthouse score 95+ on Performance, Accessibility, Best Practices.

---

## Phase 8 — Deploy & Ship

**Goal:** Vercel for the frontend, Railway for the backend, env vars configured, README written, link from portfolio.

**Prompt:**
```
Deploy the application end-to-end and write the project's public-facing docs.

1. Backend on Railway:
   - Connect the GitHub repo, set root directory to apps/api/
   - Build: Railway auto-detects pyproject.toml. If it doesn't, set:
       Build: pip install -e .
       Start: uvicorn code_council.server:app --host 0.0.0.0 --port $PORT --workers 1
   - Environment variables (set in Railway dashboard, not in repo):
       GEMINI_API_KEY, DEEPSEEK_API_KEY, MERCURY_API_KEY, Kimi_API_KEY,
       ALLOWED_ORIGINS=https://<vercel-url>
     (4 providers active. No OpenAI or Anthropic.)
   - Generate a domain in Railway. Note the URL (e.g.,
     code-council-api.up.railway.app).
   - Verify: curl https://<railway-url>/health returns {"status":"ok"}

2. Frontend on Vercel:
   - Import GitHub repo, root directory apps/web/
   - Framework preset: Next.js (auto-detected)
   - Environment variables:
       NEXT_PUBLIC_API_URL=https://<railway-url>
   - Deploy. Note the production URL.
   - Go back to Railway and add the Vercel URL to ALLOWED_ORIGINS.
   - Redeploy backend to pick up the new CORS origin.

3. Smoke test the live deployment:
   - Open the Vercel URL in an incognito window
   - /models should show 4 green dots (Gemini, DeepSeek, Mercury, Kimi)
   - Run a Solo analysis end-to-end with each model
   - Run a Council analysis with all 4 models
   - Drop an image (verifies Gemini + Kimi vision)

4. README.md at repo root. Sections, in order:
   - Title + one-line tagline ("Code Council — see how frontier models reason
     about your code")
   - Hero screenshot (just take one of the council view in action, dark)
   - "What this is" — three sentences max. Lean into the differentiator.
   - "Why this exists" — the multi-model-disagreement insight, two paragraphs.
   - "Stack" — bulleted: Next.js / FastAPI / Railway / Vercel / [model providers]
   - "Run locally" — three commands: clone, set .env, docker compose up
     (write a top-level docker-compose.yml that runs both apps for local dev)
   - "Architecture" — link to /docs/ARCHITECTURE.md, do not duplicate content
   - "What's in /legacy" — one paragraph explaining the archived modules. This
     is portfolio signal: tells reviewers you iterated and made deliberate cuts.
   - License (MIT)

5. Add a /about route in the frontend with the same content as the README's
   "What this is" + "Why this exists" sections, plus a link to the GitHub repo.
   Wire it from a small "about" link in the bottom status bar.

6. Update your portfolio site to point at the new Vercel URL with a fresh
   screenshot. Update the resume bullet — the relevant rewrite:

      "Code Council — Multi-Model Code Analysis Platform
       Differentiated from PR-automation tools by surfacing disagreement
       between frontier models (DeepSeek, Gemini, Mercury, Kimi) on the same
       code. Next.js + FastAPI, deployed Vercel + Railway, real-time SSE
       streaming, vision-capable. Live: <url> · Repo: <url>"

7. Take down the old Hugging Face deployment. Add a note to its README pointing
   at the new URL.
```

**Acceptance:** Production URL works in incognito. Every feature works end-to-end. README reads like a real product, not a homework assignment. The old Hugging Face URL is either redirected or shut down.

---

## Notes on Working Through These Phases

- **Order matters.** Phase 0 → 1 → 2 → 3 are sequential. Phases 4 and 5 can be parallelized in theory but in practice you want Solo working before Council. Phases 6, 7, 8 are sequential.
- **Don't skip Phase 3.** Locking the theme tokens before any feature work is the single biggest leverage point in this plan. Skipping it leads to drift you'll spend the rest of the project fighting.
- **The Council view (Phase 5) is the thing.** If you're tight on time, that and the refined Matrix theme are what make this not-a-clone. Everything else is competent table stakes.
- **Resist scope creep.** No auth. No saved sessions. No team workspaces. No PR integration. None of it. The product is a destination tool for individual exploration; that's the whole pitch.
