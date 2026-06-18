# Code Council — Tribunal Mode · Build Plan

> **Hackathon:** Band of Agents (June 12–19, 2026), Track 2 — Multi-Agent Software Development.
> **What this document is:** a phased, paste-into-Codex build plan that evolves the existing Code Council repo into the flagship **Tribunal** experience. Each phase is self-contained: goal, the SOTA-gap it exploits, what to build, acceptance criteria, and a Codex prompt block.
> **What this document is NOT:** a plan to out-analyze frontier code reviewers. We do not compete on code-reading horsepower. We win on a question they cannot ask.

---

## 0. The thesis (read once, keep in mind for every phase)

Frontier code reviewers (Greptile, Devin, Codex, CodeRabbit) sit **inside the diff**. They answer *"is this code correct?"* They cannot answer *"is this the change that was asked for?"* because the intent lives outside the repo — in the ticket, the spec, the policy, the design doc. This limitation now has a name in the literature: the **intent ceiling** (a structural analysis tool reads what code *does*, not what it was *supposed* to do). As of mid-2026 the tooling to cross it is openly described as not existing in production form.

**Tribunal crosses the intent ceiling by being multi-agent.** No single agent holds intent + code + tests + policy. So we coordinate specialists through **Band**, each owning one lens, and reconcile them into a verdict. This is not a workaround for the hackathon's "use Band" requirement — the coordination *is* the product. A single model cannot produce this verdict; a band of agents can.

**The three lenses frontier tools structurally miss:**
1. **Omission** (GHOST) — what was asked but is *missing*. Negative space.
2. **Scope drift** (DRIFT) — what was done but *nobody asked for*. Silent changes.
3. **Intent-anchored constraint** (WARDEN) — does it break a rule *given what it's trying to do*.

**The output artifacts that beat an opaque frontier verdict:**
- **Verdict** — `CONFORMS` / `CONFORMS_WITH_CONDITIONS` / `DOES_NOT_CONFORM`.
- **Trust Score** — 0–100. The "can I merge this AI diff without reading all 600 lines?" number.
- **Traceability Ledger** — `intent_item ↔ code ↔ test ↔ policy ↔ decision`. The SOC 2 / EU AI Act audit artifact.
- **Deliberation Transcript** — the live Band room. This is the "make collaboration visible" judging gold.

**Daily-user framing (keep the copy human):** *You asked your AI agent for X. It gave you a big diff and said "done." Tribunal convenes a panel that checks whether it actually did X — nothing missing, nothing extra, no rule broken — and shows you the argument, not just a stamp.*

---

## 1. The cast (Band agents = personas)

Each agent is a Band agent with a distinct, terse persona voice (Phase 7). Personas are functional: one agent = one lens, so the transcript is scannable. Keep them professional, not cartoonish.

| Agent | Role | Lens | Suggested model backend | Recruited |
|---|---|---|---|---|
| **CLERK** | Orchestrator | Procedure: reads docket, summons witnesses, keeps order | Fast frontier (Gemini / GPT-5.x via AI/ML API) | Always |
| **ADVOCATE** | Intent witness | What was *asked* → requirement checklist | Reasoning (Claude via AI/ML API) | Always |
| **SURVEYOR** | Implementation witness | What was *done* → uses analyzer + scanners as tools | Code model (DeepSeek) | Always |
| **GHOST** | Omission auditor | Asked-but-**missing** | Reasoning | Always |
| **DRIFT** | Scope auditor | Done-but-**not-asked** | Reasoning (open model via Featherless) | Always |
| **WARDEN** | Constraint witness | Rule/CVE/license/data-residency | Perplexity Sonar (live) + reasoning | **Only when the change touches a constrained domain** |
| **ARBITER** | Judge | Verdict + Trust Score + Ledger | Strongest reasoning (Opus-class) | Always |

Minimum to satisfy the rules is 3 agents collaborating through Band; a normal trial runs 5–7. WARDEN's conditional recruitment is the showcase for Band's "recruit a peer mid-workflow" primitive.

**Model-routing intent (also a partner-prize play):**
- Route frontier models through **AI/ML API** (partner prize: $1,000 cash + $1,000 credits).
- Route at least one open-source agent (DRIFT) through **Featherless AI** (partner prize).
- Route WARDEN's CVE lookup through the **Perplexity Agent API** (`web_search` + `fetch_url`).
- The model diversity across agents *is* your "cross-framework" evidence for the Application-of-Technology criterion.

---

## 2. What we reuse from the current repo (don't rebuild these)

The existing monorepo is `apps/web` (Next.js 15 / Vercel) + `apps/api` (FastAPI / Railway), with a `code_council` Python package and a refined-Matrix UI. Reuse map:

| Existing asset | Becomes |
|---|---|
| `apps/api/code_council/analyzer.py` | **SURVEYOR's** primary tool (reads the diff) |
| `apps/api/code_council/scanners/security.py`, `performance.py` | Tools for **SURVEYOR** and **WARDEN** |
| `apps/api/code_council/fixes.py` | Optional remediation block in the verdict |
| `apps/api/code_council/github.py` | Fetch the diff + the linked issue (Phase 1) |
| `apps/api/code_council/server.py` SSE infra | Bridge: Band room events → Next.js war room |
| `apps/web` council view (`page.tsx`) | **The War Room** (live transcript + verdict + ledger) |
| `matrix-rain.tsx`, theme tokens, score-meter | Kept — the persona/courtroom aesthetic |
| Solo/Council analyze endpoints | Kept as a secondary "quick look" mode; Tribunal is the flagship |

Nothing is thrown away. The "council of models" you already built becomes a "tribunal of role-agents."

---

## PHASE 0 — Band foundations & the Trial Protocol

**Goal:** Connect to Band, register agents, and define the Trial Protocol (room lifecycle, message/event schemas, docket and verdict shapes). No tribunal logic yet — just the rails.

**SOTA-gap note:** N/A (infrastructure). The thing to get right is using Band's **Events** stream (tool-calls/thoughts/errors) for evidence, not just text messages — most teams will only use chat messages and lose the "visible coordination" points.

**Build:**
```
Connect the repo to Band and define the Trial Protocol.

1. Add a new backend package apps/api/tribunal/ alongside code_council/.
2. Install the Band SDK (Python). Add BAND_API_KEY to .env.example and the Railway/Vercel env docs.
3. Create tribunal/band_client.py: a thin async wrapper over the Band Agent API that can
   (a) connect an agent and run its event loop (await agent.run()),
   (b) create/join a chat room,
   (c) send an @mentioned message,
   (d) post a structured Event (tool_call / finding / thought / error),
   (e) recruit a peer into the room (POST participants).
   Use the WebSocket subscription for inbound; REST for outbound.
4. Define tribunal/protocol.py with typed schemas (pydantic):
   - Docket { change_id, diff, intent_sources[], constraints[], touched_domains[] }
   - RequirementItem { id, text, source_ref, must_or_should }
   - Finding { agent, kind, ref, detail, severity, evidence_url? }
   - Verdict { state, trust_score, conditions[], ledger[], summary }
   - LedgerRow { intent_item_id, code_refs[], test_refs[], policy_refs[], decision }
5. Define the room message conventions: every witness posts findings as Band EVENTS
   (structured), and posts a one-line human-readable summary as an @mentioned MESSAGE
   to CLERK. This dual-channel is deliberate: events = machine record + audit, messages
   = legible transcript.
6. Stand up a smoke test: register 3 throwaway agents (A, B, C), open a room,
   A @mentions B, B posts an event and @mentions C, C @mentions CLERK. Print the
   transcript and the event log.

Constraints: single-host, free-tier Band (no paid memory / no distributed mode).
Do NOT build tribunal reasoning yet. End state: agents can talk, post events, and recruit.
```

**Acceptance:** Three agents join a room, exchange `@mention`-directed messages, post and read structured events, and one recruits another. Transcript + event log print cleanly.

---

## PHASE 1 — The Docket (intent ingestion)

**Goal:** Turn "a change + its intent" into a structured **Docket**. Fixtures for speed; one real source wired for credibility.

**SOTA-gap note:** This is the step that crosses the intent ceiling — frontier reviewers never ingest the intent source. Reviewing AI code without the ticket open is the named failure; *"have the original requirement or ticket open — this is non-negotiable."*

**Build:**
```
Build the Docket assembler in apps/api/tribunal/docket.py.

Inputs (any subset; at least diff + one intent source):
  - A unified git diff (paste, or fetched via code_council/github.py from a PR/compare URL).
  - Intent sources: a GitHub issue URL (fetch the title+body via github.py), a pasted
    PRD/spec snippet, and/or a policy doc snippet.
  - Constraints: a small fixtures/policies/ directory holding 3-4 example policies
    (security_policy.md, license_policy.md, data_residency.md) as markdown.

Output: a Docket object (protocol.py).
  - Parse the diff into per-file hunks; record touched paths and detect touched_domains
    via simple heuristics (deps file changed -> "dependencies"; auth/* or *security* ->
    "security"; data/db/migrations -> "data"). touched_domains drives WARDEN recruitment later.
  - Keep intent_sources as raw text + a source_ref so the ledger can cite them.

Wire ONE real source end-to-end: given a public GitHub issue URL, fetch and include it.
Everything else can be fixtures for the demo.

Acceptance: `POST /tribunal/docket` with {diff, issue_url, policy_refs} returns a Docket JSON
with touched_domains correctly inferred on the three demo fixtures.
```

**Acceptance:** Given a diff + a GitHub issue URL (+ optional policy refs), returns a structured Docket with correct `touched_domains`.

---

## PHASE 2 — Core witnesses (CLERK, ADVOCATE, SURVEYOR)

**Goal:** A trial runs end-to-end with the three always-on agents. CLERK opens the room and posts the docket; ADVOCATE produces the requirement checklist; SURVEYOR produces the implementation report.

**SOTA-gap note:** ADVOCATE's explicit, itemized requirement checklist is the artifact frontier tools lack — it makes intent *machine-comparable*. SURVEYOR reuses your analyzer so we don't reinvent code reading; we only need "what does the diff do," not "is it world-class code."

**Build:**
```
Implement three Band agents in apps/api/tribunal/agents/.

CLERK (clerk.py):
  - On a new trial, create a Band room, post the Docket summary as the opening message,
    and @mention ADVOCATE and SURVEYOR to begin. Track which witnesses have reported.
  - For now, summon ADVOCATE and SURVEYOR unconditionally. (WARDEN/GHOST/DRIFT in later phases.)

ADVOCATE (advocate.py):
  - Input: Docket.intent_sources. Output: an ordered RequirementItem[] checklist, each item
    atomic, id'd (R1, R2, ...), tagged must/should, with a source_ref back to the intent text.
  - Post each item as a Band EVENT (kind="requirement") and a compact checklist MESSAGE to CLERK.

SURVEYOR (surveyor.py):
  - Input: Docket.diff. Tools: code_council analyzer + security/performance scanners.
  - Output: an ImplementationFinding[] describing what the diff actually does, per hunk,
    mapped to file:line refs. Post as EVENTS (kind="implementation") + a summary MESSAGE.

Orchestrate a single trial in tribunal/trial.py: open room -> CLERK -> ADVOCATE + SURVEYOR
(in parallel) -> collect events. Expose `POST /tribunal/run` (SSE) that streams room
messages+events to the caller (reuse the SSE pattern from code_council/server.py).

Acceptance: running a trial on the fixtures yields a transcript containing a clean,
itemized requirement checklist and a per-hunk implementation report, both visible as
Band events and as legible messages.
```

**Acceptance:** End-to-end trial with 3 agents; transcript shows an itemized checklist and a per-hunk implementation report.

---

## PHASE 3 — The novel lenses (GHOST omission + DRIFT scope)

**Goal:** The two agents that make this not-a-PR-bot. GHOST finds asked-but-missing; DRIFT finds done-but-not-asked.

**SOTA-gap note:** This is the negative-space detection structural tools cannot do. They review what is present for correctness; they do not reconcile presence against an intent checklist. GHOST and DRIFT operate on the *delta between ADVOCATE and SURVEYOR* — an artifact that only exists because we ran two specialist agents and coordinated them through Band. **Build the strongest fixture here:** an AI-generated diff that silently omits one required item and adds one unrequested change. If GHOST and DRIFT each catch their target, the whole thesis is proven on screen.

**Build:**
```
Implement GHOST and DRIFT as Band agents recruited by CLERK after ADVOCATE and SURVEYOR report.

GHOST (ghost.py) — omission auditor:
  - Input: ADVOCATE's RequirementItem[] + SURVEYOR's ImplementationFinding[].
  - For each requirement, decide MET / PARTIAL / UNMET with the implementation ref that
    satisfies it (or none). Output Finding[] (kind="omission") for every UNMET/PARTIAL item.
  - Voice: states the missing item and what evidence it expected to find but didn't.

DRIFT (drift.py) — scope auditor:
  - Input: same two sets.
  - For each implementation finding, decide whether any requirement authorizes it.
    Output Finding[] (kind="scope_drift") for changes with no authorizing requirement,
    flagged risk-tiered (cosmetic refactor vs behavior change vs new dependency).
  - Voice: names the unauthorized change and the docket gap.

CLERK now waits for ADVOCATE+SURVEYOR, then @mentions GHOST and DRIFT with both sets.

Build fixture B (the money demo): a diff that implements 4 of 5 requirements (omits rate
limiting) AND adds an unrequested change to an auth middleware. Assert in a test that GHOST
emits an omission for R-rate-limit and DRIFT emits a scope_drift for the auth change.

Acceptance: on fixture B, GHOST flags exactly the omitted requirement and DRIFT flags exactly
the unrequested change; both appear in the transcript as events with evidence refs.
```

**Acceptance:** On the omission+creep fixture, GHOST flags the missing requirement and DRIFT flags the unrequested change, each with evidence refs.

---

## PHASE 4 — The Warden (constraint + live CVE via Perplexity)

**Goal:** A constraint specialist recruited only when the change touches a constrained domain. Live CVE/advisory lookup makes it concrete and impressive.

**SOTA-gap note:** Conditional recruitment is Band's signature primitive ("bring in a specialist mid-workflow"); most teams won't use it. Live CVE verification (find the official vendor advisory + patched version for a dependency bump) is a task Perplexity's Agent API is documented to do well. This is also the intent-anchored constraint check: the rule matters *because of what the change does*.

**Build:**
```
Implement WARDEN (warden.py), recruited by CLERK ONLY when Docket.touched_domains intersects
{dependencies, security, data}.

WARDEN tools:
  - For dependency changes: call the Perplexity Agent API (model perplexity/sonar) with
    tools [web_search, fetch_url] to find CVEs + official vendor advisory + patched version
    for the changed package/version. Wrap in tribunal/tools/cve_lookup.py. Read PERPLEXITY_API_KEY.
  - For security/data domains: run code_council security scanner + check the relevant policy
    fixture (e.g., data_residency.md) for a violation.
  - Post each result as a Band EVENT (kind="constraint", with evidence_url for advisories) and
    a summary MESSAGE to CLERK.

CLERK recruitment: after the docket is posted, if a constrained domain is present, CLERK calls
the Band "recruit peer" API to add WARDEN to the room, then @mentions it. This recruitment
must be visible in the transcript ("CLERK summons WARDEN re: dependency change").

Build fixture C: a dependency bump (e.g., a package to a version with a known advisory). Assert
WARDEN is recruited, fetches an advisory (live or recorded), and posts a constraint finding.

Acceptance: a dependency-touching trial recruits WARDEN on-screen, WARDEN returns a real
advisory reference, and a constraint finding lands in the transcript.
```

**Acceptance:** Dependency change → CLERK recruits WARDEN live → WARDEN fetches a real advisory → constraint finding in the transcript.

---

## PHASE 5 — The Arbiter (verdict, Trust Score, ledger)

**Goal:** Turn the deliberation into a binding, structured ruling.

**SOTA-gap note:** The Trust Score answers the daily question ("can I merge this without reading it all?"); the Ledger is the compliance/audit artifact (SOC 2 / EU AI Act traceability) that frontier reviewers don't emit. Both are *reconciliations across agents* — only possible because we coordinated specialists.

**Build:**
```
Implement ARBITER (arbiter.py). Input: all events (requirements, implementation, omission,
scope_drift, constraint). Output: a Verdict (protocol.py).

Verdict logic (deterministic scaffold + model reasoning for nuance):
  - state:
      DOES_NOT_CONFORM if any MUST requirement is UNMET (GHOST) or any critical constraint
        is violated (WARDEN);
      CONFORMS_WITH_CONDITIONS if only SHOULD items unmet, or scope_drift present, or a
        non-critical constraint condition;
      CONFORMS otherwise.
  - trust_score (0-100): start 100; subtract weighted penalties for unmet MUST (-30 each),
    unmet SHOULD (-8), behavior-changing scope_drift (-15), cosmetic drift (-3), critical
    constraint (-40), advisory constraint (-10). Floor 0. Show the math in the verdict.
  - conditions[]: concrete, actionable (e.g., "Implement R3 (rate limiting)", "Remove or
    document the auth middleware change", "Pin <pkg> to >= patched_version").
  - ledger[]: one LedgerRow per requirement item, citing code refs (SURVEYOR), test refs
    (if any), policy refs (WARDEN), and the per-item decision (MET/UNMET/CONDITION).
  - summary: 2-3 sentence ruling in ARBITER's voice.

Post the Verdict as a final Band EVENT (kind="verdict") and a MESSAGE.

Acceptance: each of the 3 fixtures yields the expected state (A: CONFORMS, B: DOES_NOT_CONFORM,
C: CONFORMS_WITH_CONDITIONS) with a sensible trust score and a complete ledger.
```

**Acceptance:** The three fixtures produce the expected verdict states, sensible trust scores, and complete ledgers.

---

## PHASE 6 — The War Room (Next.js UI over the Band room)

**Goal:** The human's window. Repurpose the existing council view into a live courtroom that streams the deliberation and renders the verdict + ledger.

**SOTA-gap note:** Visible deliberation is the differentiator vs an opaque frontier verdict, and it directly serves the Presentation judging criterion. You already built a multi-verdict view; this is its evolution.

**Build:**
```
Repurpose apps/web council view into the War Room (apps/web/src/app/page.tsx, Tribunal tab).

Bridge: extend apps/api server to expose `GET /tribunal/stream/{trial_id}` (SSE) that relays
the Band room's messages+events (CLERK..ARBITER) to the browser, reusing the existing SSE client.

Layout:
  - LEFT: the Docket — the diff (Monaco, read-only) + intent sources + detected domains.
  - CENTER: the Deliberation — a live transcript with one lane/color per persona (use each
    agent's provider color you already have). Show recruitment moments explicitly
    ("CLERK summons WARDEN"). Render Band EVENTS as compact evidence chips under each message.
  - RIGHT: the Ruling — appears when ARBITER posts:
      * Verdict stamp (CONFORMS / WITH CONDITIONS / DOES NOT CONFORM) — big, monospace.
      * Trust Score meter (reuse the score-meter component, animate 0 -> score).
      * Conditions list.
      * Traceability Ledger table (intent item -> code -> test -> policy -> decision).

Keep the refined-Matrix theme: two-greens, JetBrains Mono, subtle matrix rain. The persona
lanes use the provider colors. Add an "Export ledger" button (JSON + printable view).

Empty state: "Submit a change and its ticket to convene the Tribunal."

Acceptance: paste a diff + issue URL, click Convene, watch the trial stream in live with
persona lanes and recruitment moments, then see the verdict stamp, trust meter, and ledger.
```

**Acceptance:** Submit a diff + issue URL → watch the trial stream live with persona lanes → see verdict stamp, trust meter, and ledger.

---

## PHASE 7 — Persona voice, fixtures, audit export

**Goal:** The 10% that makes it land on video and in a portfolio.

**Build:**
```
1. Persona voices: give each agent a short system-prompt persona so the transcript is
   instantly legible and characterful but professional. Guidance:
     CLERK    - procedural, neutral, brief ("Convening. Docket #... Summoning ADVOCATE, SURVEYOR.")
     ADVOCATE - precise, itemizing ("The petition requires: R1..., R2...")
     SURVEYOR - dry, factual, ref-heavy ("auth.py:42 adds a bcrypt hash. No rate limit found.")
     GHOST    - spare, names absences ("R3 unmet. Expected a limiter; none present.")
     DRIFT    - watchful ("middleware/auth.py changed. No docket item authorizes this.")
     WARDEN   - rule-bound, cites sources ("CVE-... affects <pkg> < x.y.z. Advisory: <url>.")
     ARBITER  - final, measured ("Ruling: DOES NOT CONFORM. Trust 41/100. Conditions: ...")
   Keep each voice to a recognizable cadence; do not overdo it.

2. Three polished demo fixtures (fixtures/cases/):
     A) Faithful change -> CONFORMS, high trust.
     B) AI diff with a silent omission + scope creep -> DOES NOT CONFORM. (The headline demo.)
     C) Dependency bump with a real advisory -> CONFORMS WITH CONDITIONS.

3. Audit export: serialize the Verdict + Ledger + full event log into an audit artifact
   (JSON + a printable HTML view), framed as the change's traceability record. This is the
   enterprise/compliance hook (SOC 2 / EU AI Act traceability).

Acceptance: all three fixtures run cleanly with legible persona transcripts; the audit
artifact exports and reads like a real compliance record.
```

**Acceptance:** Three fixtures run with legible persona transcripts; audit artifact exports cleanly.

---

## PHASE 8 — Submission

**Goal:** Package for the judges and the portfolio.

**Build:**
```
1. Demo video (<= 5 min, <= 300MB) script:
   0:00 The intent ceiling — frontier reviewers read code, not intent. One sentence + a visual.
   0:30 The daily pain — "you asked your agent for X, got a 600-line diff that says 'done'."
   0:50 Convene the Tribunal on fixture B. Watch CLERK summon witnesses through Band.
   2:00 GHOST catches the omission; DRIFT catches the scope creep; CLERK recruits WARDEN live.
   3:10 ARBITER rules: DOES NOT CONFORM, Trust 41, with the ledger.
   3:50 Show the Band room view — this verdict is impossible without coordinated agents.
   4:20 Enterprise framing: the ledger is an audit trail. Close.

2. Repo: clean README (problem -> Tribunal -> Band architecture -> run locally), ARCHITECTURE.md
   updated with the agent topology + Trial Protocol, /legacy preserved, MIT license.

3. The four rubric-aligned writeups (paste into the submission fields):
   - Application of Technology: the verdict is impossible without Band coordination; CLERK
     recruits witnesses on demand; witnesses post structured Events; @mention routing directs
     the deliberation; cross-model agents = cross-framework.
   - Presentation: the War Room makes the multi-agent workflow legible — personas, recruitment,
     evidence chips, verdict, ledger.
   - Business Value: crosses the intent ceiling; solves the daily AI-diff-trust problem and the
     enterprise audit-trail problem (SOC 2 / EU AI Act traceability).
   - Originality: not PR review — intent-conformance adjudication with omission + scope-drift
     (negative-space) detection and a transparent, recruited tribunal. Beyond Codeband
     (correctness) and beyond spec-driven generators (opaque, single-vendor).

4. Partner-tech notes: AI/ML API (frontier agents), Featherless (open-source agent), Perplexity
   Agent API (WARDEN CVE lookup). Claim the AI/ML API and Featherless partner prizes.

Acceptance: submission package complete; video tells the intent-ceiling story end to end.
```

**Acceptance:** Submission package complete; the video tells the intent-ceiling story with a live trial.

---

## Build order & parallelism

`0 → 1 → 2 → 3 → 5 → 6` is the critical path that produces a working demo. **4 (WARDEN)** and **7 (polish)** can slot in once 2–3 are stable. If time collapses, the minimum winning demo is **0,1,2,3,5,6** on fixture B — that alone proves the thesis (omission + scope-drift caught by coordinated agents, with a verdict). Add WARDEN if at all possible; the live CVE recruitment is the single most impressive on-camera moment.

## Free-tier & risk register

- **Band free tier:** single host only; no paid memory / distributed mode. Run all agents in one process; keep to ≤7 agents. Minimum is 3.
- **Time box (7 days, you're busy):** fixtures for everything except one real GitHub issue fetch and one real CVE lookup. Do not build real Jira/Linear integrations now.
- **Perplexity sandbox uncertainty:** the full "Search as Code" sandbox may be internal; the `web_search` + `fetch_url` tools on the Agent API are sufficient for WARDEN. Have a recorded advisory as a fallback for the video so a live API hiccup can't break the demo.
- **Scope discipline:** no auth, no multi-tenant, no persistence beyond the audit export. Tribunal on a diff is the whole pitch.
- **Don't drift into Codeband:** if you find yourself building "agents that write/merge code," stop — that's the reference impl. We adjudicate faithfulness; we do not author code.

## Naming

Umbrella product stays **Code Council**. Flagship mode is **Tribunal**. If you want a sharper standalone name for the portfolio, candidates: *Intent Tribunal*, *Faithful*, *Concord*, *The Reckoning*. Pick one and use it consistently across repo, UI, video, and submission.
