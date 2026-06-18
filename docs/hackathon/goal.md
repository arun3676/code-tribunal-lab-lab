# Code Council Tribunal — Goal Document

> **Hackathon:** [Band of Agents Hackathon](https://lablab.ai) · June 12–19, 2026  
> **Track:** Track 2 — Multi-Agent Software Development  
> **Repo:** [code-tribunal-lab-lab](https://github.com/arun3676/code-tribunal-lab-lab) (Code Council monorepo)  
> **Deadline:** June 19, 2026 · 8:00 AM PDT (15:00 UTC)

---

## North Star

**Build and submit Code Council Tribunal** — a Band-powered intent-conformance court for AI-generated code that proves whether a diff actually matches what was asked for, not just whether the code looks correct.

**One-line pitch:**

> Did the AI build what you actually asked for?

**Repeat line for judges:**

> Not code review. **Intent-conformance review.**

---

## What We Are Building

Code Council Tribunal evolves the existing **Code Council** repo (multi-model code analysis sandbox) into a **Band-native multi-agent merge-prep workflow**.

When an AI coding agent returns a large diff and says "done," Tribunal:

1. Opens a **Band room** (CLERK orchestrates)
2. Extracts **requirements** from the ticket/spec (ADVOCATE)
3. Inspects **what the diff actually changed** (SURVEYOR)
4. Finds **requested work that is missing** (GHOST — negative space)
5. Finds **unauthorized scope changes** (DRIFT — scope creep)
6. **Recruits** a security witness when auth/security is touched (WARDEN)
7. Issues a **merge verdict**, trust score, and traceability ledger (ARBITER)

The **Band deliberation transcript** is the primary artifact — visible coordination, not a black-box verdict.

---

## Why This Problem Matters (Business Value)

| Pain | Tribunal answer |
|------|-----------------|
| AI agents produce 600-line diffs from one sentence | Reconcile diff against original intent before merge |
| Humans cannot review every line confidently | Trust Score (0–100): "Can I merge without reading it all?" |
| Standard PR bots only ask "is the code correct?" | Ask "is this the change that was **requested**?" |
| Enterprise needs audit trails for AI-assisted code | Traceability ledger: intent item ↔ code ↔ decision |
| Silent omissions and scope creep in AI diffs | GHOST + DRIFT — failures normal diff review misses |

**Enterprise framing:** Pre-merge trust layer for teams adopting Cursor, Copilot, Codex, Devin, and other AI coding agents. Ledger suitable for SOC 2 / EU AI Act traceability narratives.

---

## Hackathon Alignment

### Hard requirements (must pass or disqualified)

| Requirement | How Tribunal satisfies it |
|-------------|---------------------------|
| ≥3 agents collaborate **through Band** | 6–7 agents: CLERK, ADVOCATE, SURVEYOR, GHOST, DRIFT, ARBITER (+ WARDEN) |
| Band = **actual collaboration layer** | @mention routing, structured events, mid-trial recruitment — not a notification wrapper |
| Cross-framework multi-agent system | Different model backends per agent (AI/ML API, Featherless, Code Council analyzer) |
| Original, MIT-compliant work | New Band tribunal layer on existing Code Council repo; Band integration is new work |
| Public GitHub + demo URL + video | Vercel (web) + Railway (api) + ≤5 min video |

### Track 2 fit

Official Track 2 examples include: planner/engineer/reviewer/tester workflows, cross-model code review, automated PR review and merge preparation, QA, documentation, release coordination.

Tribunal maps to **review + merge preparation** with a novel wedge: **intent-conformance adjudication** instead of generic bug-finding.

### Judging criteria — what judges score and our answer

| Criterion | Weight | What judges look for | Tribunal demo moment |
|-----------|--------|----------------------|----------------------|
| **Application of Technology** | Highest | Band coordinates specialists; clear handoffs, shared context, role specialization, task state | CLERK `@ADVOCATE` `@SURVEYOR` → structured events → CLERK `@GHOST` `@DRIFT` → CLERK recruits WARDEN → ARBITER verdict from event log |
| **Presentation** | High | Problem, agent roles, Band's role, context flow, enterprise value — legible in first 60 seconds | War Room UI: persona lanes, @mentions, evidence chips, recruitment banner, verdict stamp |
| **Business Value** | High | Real enterprise workflow; reduces manual coordination; improves decisions | "AI said done but omitted rate limiting and changed auth middleware" — BLOCK with conditions |
| **Originality** | High | Beyond chatbot / single-agent / linear automation | GHOST (negative space) + DRIFT (unauthorized changes) — impossible with single-model diff review |

**Kickoff guidance (from stream):** *"Make the collaboration visible. When a judge looks at your project, they should see right away which agent did what and where the handoffs happened."*

---

## Differentiation vs Competitors

| Submission | Their story | Our counter |
|------------|-------------|-------------|
| **AutoReview Crew** (closest) | 4 agents review PR for correctness/security/tests | We verify PR **matches the original request** — GHOST catches omissions, DRIFT catches scope creep |
| **Codeband** (reference impl) | Planner + coder + reviewer + mergemaster | We **adjudicate** AI code; we do not write or merge code |
| **DevBand** | Plan → implement → test → review → document | We focus on **intent reconciliation** before merge, not full delivery pipeline |
| **AetherDev Pro** | Multi-agent software assembly IDE | Courtroom adjudication, not IDE |
| **MUSTER** | Incident war-room with specialist recruitment | Same recruitment pattern, applied to **merge-prep / intent gaps** |
| **Council (LLM Council pattern)** | Models debate | Specialized **roles** produce traceable merge verdict + ledger |

**Winning wedge:** Frontier reviewers sit *inside* the diff. Tribunal crosses the **intent ceiling** — intent lives in the ticket/spec, outside the repo.

---

## Agent Cast

| Agent | Role | Lens | Provider | Recruited |
|-------|------|------|----------|-----------|
| **CLERK** | Orchestrator | Procedure, summons witnesses | Band | Always |
| **ADVOCATE** | Intent witness | What was *asked* → requirement checklist | **AI/ML API** | Always |
| **SURVEYOR** | Implementation witness | What was *done* → diff analysis | Code Council analyzer | Always |
| **GHOST** | Omission auditor | Asked-but-**missing** | **AI/ML API** | After ADVOCATE + SURVEYOR |
| **DRIFT** | Scope auditor | Done-but-**not-asked** | **Featherless** | After ADVOCATE + SURVEYOR |
| **WARDEN** | Constraint witness | Security/policy for constrained domains | Band + policy | **Recruited live** when auth/security touched |
| **ARBITER** | Judge | Verdict + Trust Score + Ledger | **AI/ML API** | Always |

Minimum hackathon bar: 3 agents. **Competitive demo: 6–7** with visible recruitment.

---

## Partner & Sponsor Goals

| Partner | Prize / benefit | Our usage | Visibility requirement |
|---------|-----------------|-----------|------------------------|
| **Band** | Core platform; BANDHACK26 = 1 month Pro free | Rooms, @mentions, events, recruitment | Transcript is the demo; show real Band room in video |
| **AI/ML API** | Best use: $1,000 cash + $1,000 credits | ADVOCATE, GHOST, ARBITER | UI badges + README + video narration + tags |
| **Featherless** | 1st/2nd/3rd partner prizes (inference credits + Claw Pro) | DRIFT scope auditor | UI badge + README + video; promo **BOA26** |

Claim AI/ML API coupon through lablab.ai ($10/person). Featherless: setup guide + BOA26 code.

---

## Success Definition

### Minimum viable submission (must ship)

- [ ] `/tribunal` route loads on deployed app
- [ ] One demo fixture: auth login ticket + diff with omission + scope creep
- [ ] Real Band room with @mention handoffs (not fake transcript only)
- [ ] ≥5 agents visible in deliberation
- [ ] GHOST catches missing rate limiting (R3)
- [ ] DRIFT catches unauthorized auth middleware change
- [ ] ARBITER: `DOES_NOT_CONFORM`, trust score, `BLOCK`, ledger
- [ ] Sponsor badges visible (AI/ML API, Featherless)
- [ ] Public GitHub repo, demo URL, video ≤5 min, slides, cover image
- [ ] Submitted on lablab.ai before deadline

### Competitive submission (target)

Everything above, plus:

- [ ] WARDEN recruited live mid-trial (Band `add participant` visible)
- [ ] Dual-channel output: human messages + structured events (evidence chips)
- [ ] At least one live AI/ML API call + one live Featherless call (partner prize eligibility)
- [ ] Band room shown **side-by-side** with War Room UI in video
- [ ] First 60 seconds show problem → @mentions → GHOST/DRIFT findings
- [ ] Traceability ledger maps R1–R5 → MET/UNMET/DRIFT/CONDITION
- [ ] README rewritten for Tribunal (not "model comparison sandbox")

### Stretch (only if time remains)

- Second fixture (faithful change → CONFORMS)
- Audit export (JSON/HTML)
- GitHub issue URL ingestion
- Perplexity CVE lookup for WARDEN (7-day plan; cut for 1-day)

---

## Non-Goals (Explicitly Out of Scope)

Do **not** build these — they hurt a one-day submission:

- Autonomous coding agent / auto-merge
- Full GitHub PR ingestion pipeline
- Jira / Linear OAuth integrations
- Persistent database / multi-tenant auth
- Real CVE lookup (unless WARDEN done early)
- Three polished fixtures + audit export
- Competing with Codeband on code generation
- Rebuilding the entire homepage — use standalone `/tribunal` route

---

## Constraints

| Constraint | Implication |
|------------|-------------|
| **One day** to build + submit | Deterministic staged runner; one money fixture; cut 7-day plan to critical path |
| **Busy with Cork** | Do not overbuild; demo reliability > feature breadth |
| **Repo packaging** | Tribunal code under `apps/api/code_council/tribunal/` (pyproject packages `code_council*` only) |
| **Existing app must work** | Solo/Council modes untouched; Tribunal is additive |
| **Band free/Pro tier** | Single-host process; ≤7 agents; real Band for submission demo |
| **55+ existing submissions** | Differentiation must be obvious in first 60 seconds of video |

---

## Hero Demo (Money Fixture)

**Ticket — Implement secure login:**

- R1: Add `/api/login` endpoint
- R2: Verify password using bcrypt
- R3: Rate-limit failed login attempts (5 per 15 minutes) — **MUST**
- R4: Add audit log entry for failed login
- R5: Add regression tests
- **Constraint:** Do not change existing auth middleware behavior

**Diff intentionally:**

- ✅ Implements R1, R2, R4, one test
- ❌ Omits R3 (rate limiting)
- ⚠️ Sneaks auth middleware behavior change

**Expected outcome:**

- GHOST → R3 unmet (critical omission)
- DRIFT → middleware change unauthorized (scope drift)
- CLERK recruits WARDEN (auth domain)
- ARBITER → `DOES_NOT_CONFORM`, Trust ~41/100, `BLOCK`

This is the **60-second judge moment** — normal diff review misses both failures.

---

## Submission Package (lablab.ai)

| Field | Value |
|-------|-------|
| **Title** | Code Council Tribunal |
| **Short description** | Band-powered intent-conformance review room for AI-generated code. Specialized agents compare the original ticket against the actual diff, catch omissions and scope drift, recruit security witnesses, and issue a merge verdict with a traceability ledger. |
| **Tags** | Band, Track 2, Multi-Agent Software Development, AI/ML API, Featherless, Code Review, AI Coding Agents, Developer Tools, FastAPI, Next.js, Vercel, Railway |
| **Cover image** | War Room screenshot with verdict stamp |
| **Video** | ≤5 min, ≤300MB — problem → Band handoffs → GHOST/DRIFT → WARDEN recruitment → verdict |
| **Slides** | Problem, architecture, demo screenshots, rubric alignment |
| **GitHub** | Public repo (this monorepo) |
| **Demo URL** | Vercel frontend → Railway API |
| **Platform** | Vercel + Railway |

---

## Portfolio Value (Beyond Hackathon)

Even if we do not win main prizes:

- Strong AI portfolio piece: multi-agent enterprise workflow on Band
- Evolves Code Council from "model sandbox" to "enterprise merge-prep product"
- Demonstrates intent-conformance — a real gap in AI coding tooling
- Reusable for LinkedIn, GitHub profile, future enterprise pitches
- Partner credits (AI/ML API, Featherless) for continued building

---

## Related Artifacts

| File | Purpose |
|------|---------|
| [`plan.md`](./plan.md) | Execution plan — blocks, files, acceptance criteria, video script |
| [`../STRUCTURE.md`](../STRUCTURE.md) | Repo layout and conventions |
| [`../MIGRATION_PLAN.md`](../MIGRATION_PLAN.md) | Historical monorepo migration spec |

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Track 2, not Track 3 | Software delivery / merge-prep fit; borrow Track 3 traceability without leaving track |
| Tribunal, not generic PR review | AutoReview Crew and others already occupy "agents find bugs" |
| `code_council/tribunal/` not `apps/api/tribunal/` | pyproject.toml packages `code_council*` only — avoids deploy breakage |
| Standalone `/tribunal` route | Homepage has complex solo/council/multimodal state — too risky to edit |
| Deterministic runner + optional live LLMs | One-day reliability; fallbacks labeled honestly in UI |
| GHOST + DRIFT first | Hero moment; everything else supports this |
| Real Band required for submission | Hackathon explicitly rejects thin wrappers |

---

*Last updated: June 18, 2026 — pre-build context document*
