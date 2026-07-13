# ROADMAP — AI Agent Portfolio Build Order

**Eighteen projects.** The original 8 (from `AI-Agent-Project-Ideas.md`), + 4 Sonnet added for MCP authoring / RAG depth / safety / CI/CD (09–12), + 3 from the Fable-5 revision for observability / memory / computer-use (13–15), + **3 from the final gap sweep of `500-AI-Agents-Projects`** for the last genuinely-uncovered patterns: **16 Multimodal**, **17 Voice/Real-Time**, **18 Local/Offline**. Full plans, first-principles teaching notes, and verified resource lists for all 18 live in `projects/<NN-name>/`. `AUDIT.md` records the audit of the prior version; **`LEARNING-GUIDE.md` is the companion to this file** — it explains every project (what it is, real-world use, skills, where they go next) in the exact order below. This ROADMAP and LEARNING-GUIDE are one source of truth; if they ever disagree, they're both wrong.

## India-first localization (read alongside the contracts)

Every project is localized for the Indian market by default (see each PLAN.md §10 "Localization"); the **technical curriculum is unchanged** — only data sources, document types, regulators, currency, and examples are Indian. Setup impact is minimal: the new data libraries are all `pip install` (yfinance `.NS`/`.BO`, `nsepython`, `jugaad-data`, `bsedata`, AI4Bharat models, Ollama for local). Key points:
- **Project 01** swaps SEC filings → NSE/BSE annual reports/results and introduces a **ports-&-adapters data layer** so "worldwide mode" (US/global) is a config flag, not a rewrite — a portfolio-worthy engineering lesson in itself.
- **Deep-localized** (domain/data materially Indian): 01 (NSE/BSE), 02 (GST/KYC + DPDP Act), 05 (NSE bhavcopy demo; Spider benchmark unchanged), 11 (Aadhaar/PAN/UPI PII + DPDP), 16 (GST invoices/Indian docs), 17 (Hindi/Hinglish + AI4Bharat voice), 18 (DPDP data-residency + Indian open models). 04 gains IST/Indian-job-board/UPI-expense context; 10 reuses 01's Indian corpus.
- **Location-neutral by design** (stated explicitly, not forced): 03, 06, 07, 08, 09, 12, 13, 14, 15 — their patterns carry no market assumptions; India shows through only via the Indian agents they instrument/evaluate/orchestrate.

## Shared contracts (read first — what makes the set cohere)

- **Target Agent Contract.** A compliant agent exposes HTTP `/invoke` returning `{output, trajectory, version, cost_usd, latency_ms}`, where `trajectory` is the ordered tool calls with tokens/latency (or an OTel GenAI export). **Emitted by** 01, 02, 05, 08, 15, 16. **Consumed by** 03 (eval), 11 (guardrail), 12 (release gate), 17 (voice brain). **Reference implementation:** Project 13.
- **MCP Server Contract.** stdio locally, **Streamable HTTP** remotely (not deprecated SSE), Inspector-clean, typed schemas. Used by 04, 08, 09.

## The definitive build order (optimized for cumulative learning)

Ordered so each project reuses the previous ones' skills. `⟶` = hard prerequisite (needs its output/interface); `~` = soft reuse (reuses an idea/corpus). Durations are part-time (10–15 hrs/wk).

| # | Project | New skills it introduces | Reinforces | Prereqs | Dur. |
|---|---|---|---|---|---|
| 1 | **01 Financial Research Analyst** | Supervisor multi-agent, agentic/corrective RAG, reflection, tool design, deploy | — | — | 4–6 wk |
| 2 | **13 Agent Observability** | OTel GenAI tracing, cost/latency attribution, the Target Agent Contract | multi-agent internals | 01 | 3–4 wk |
| 3 | **10 RAG Architecture Bake-Off** | 4 RAG variants, controlled experiments, RAGAS | RAG, eval rigor | 01~ | 3–4 wk |
| 4 | **03 Eval & Observability Platform** | LLM-as-judge + calibration, simulated users, regression detection | evals, the Contract | 01⟶, 13~ | 4–6 wk |
| 5 | **09 MCP Server Trilogy** | MCP *authoring* + publishing (PyPI + registry), protocol compliance | API design | — | 3 wk |
| 6 | **04 MCP Personal Ops Agent** | MCP *consumer*, proactive scheduling, approval gate, memory-lite | MCP, HITL | 09⟶ | 3–4 wk |
| 7 | **14 Long-Term Memory System** | Episodic/semantic/procedural memory, consolidation/forgetting policies | memory, benchmarking | 04⟶ | 4 wk |
| 8 | **08 A2A Multi-Framework Network** | A2A protocol, framework-agnostic orchestration, MCP+A2A together | interop, MCP | 01⟶, 04~ | 3–4 wk |
| 9 | **02 Document-Processing HITL Pipeline** | Durable execution (interrupt/resume), idempotency, audit, deterministic rules | HITL, the Contract | — | 4–6 wk |
| 10 | **11 Agent Guardrail Gateway** | Safety middleware, PII redaction, injection defense, rate/spend limits | safety, the Contract | 01⟶/02⟶ | 3–4 wk |
| 11 | **12 Agent Release Gate (CI/CD)** | Eval-as-CI, reusable GitHub Action, baseline-relative regression | evals, packaging | 03⟶, 01⟶, 02⟶ | 3–4 wk |
| 12 | **16 Multimodal Document Intelligence** | Vision-language, layout-aware doc AI, visual retrieval (ColPali-style) | RAG, retrieval, citation | 10~, 01~ | 4 wk |
| 13 | **05 Self-Healing SQL Analytics Agent** | Text-to-SQL, self-correction loop, structural (read-only) safety, benchmarking | bounded retry, safety | — | 3–4 wk |
| 14 | **18 Local / Offline Open-Weight Agent** | Self-hosting, quantization, grammar-constrained decoding, hosted-vs-local analysis | structured output, benchmarking | 05⟶ (port) | 3–4 wk |
| 15 | **06 Codebase Onboarding Agent** | AST-aware chunking, code retrieval (hybrid), verified citation, static analysis | RAG, grounded citation | — | 4–5 wk |
| 16 | **15 Computer-Use / Browser Agent** | GUI perception (a11y+vision), action spaces, recovery, irreversible-action gate | vision grounding, safety | — | 4–5 wk |
| 17 | **17 Voice / Real-Time Agent** | Streaming STT/LLM/TTS, latency budgeting, barge-in, turn detection | real-time systems, reuse | 04⟶/01⟶ (brain), 13~ | 3–4 wk |
| 18 | **07 Red-Team vs Blue-Team Arena** | FSM-constrained orchestration, isolated eval labs, security metrics | orchestration, safety framing | — | 4–5 wk |

## Dependency graph

```
01 ─┬─> 13 (instruments 01; defines Target Agent Contract)
    ├─> 10 (RAG depth; reuses 01 corpus)
    ├─> 03 (evaluates 01; consumes Contract) ─> 12 (packages 03's harness)
    ├─> 08 (reuses 01's specialists over A2A)
    ├─> 11 (protects 01/02; reads Contract)
    └─> 16 (reuses 01 corpus + 10's RAG)
02 ─┬─> 11
    └─> 12 (second target repo)
09 ─> 04 (consumes 09's servers) ─┬─> 14 (memory plugs into 04)
                                   └─> 17 (voice reuses 04 as its brain)
05 ─> 18 (ports 05 to local models)
15 · 06 · 07 (standalone; 15 & 16 emit the Contract)
```

## What can be built in parallel

- After **01 + 13**, the **RAG track (10)** and the **MCP track (09)** are independent — run them in parallel if you have the bandwidth.
- **05, 06, 07** are mutually independent single-agent projects — any order, or parallel.
- **16 and 15** (vision modality vs. GUI modality) are independent of each other.
- **11 and 12** both need production targets (01/02) but are otherwise independent.
Everything else is best done in the listed order because each hard-depends on an earlier project's output or interface.

## Resume checkpoints (when you have something worth showing)

- **After #4 (03):** ✅ **First shippable portfolio.** A deployed multi-agent system (01), instrumented with traces (13), with a data-backed eval harness proving it works (03). This trio alone is a stronger portfolio than most candidates have. *Start applying here.*
- **After #8 (08):** ✅ **Protocol-fluent.** You now demonstrate MCP authoring (09), MCP consumption + memory (04, 14), and A2A interop (08) — the 2026 protocol stack most candidates lack.
- **After #11 (12):** ✅ **Production-grade.** Durable HITL (02), a reusable safety gateway (11), and eval-as-CI (12) — the "I ship agents responsibly" story.
- **After #16 (15) / #17 (17):** ✅ **Multimodal breadth.** Vision documents (16), GUI automation (15), and voice (17) — you cover every input modality, which very few candidates do.

## Timelines

| Track | Scope | Realistic timeline |
|---|---|---|
| **Full mastery** | All 18 | ~9–13 months part-time; ~5–6 months full-time |
| **Core production portfolio** | 01 + 13 + 10 + 03 + 09 + 04 + 02 | ~4–5 months part-time — build → observe → prove → author protocols → ship a durable pattern |
| **Minimum viable** | 01 + 13 + 03 | ~2–2.5 months part-time — a built, observable, *proven* agent (Resume Checkpoint 1) |
| **Fast-track** | 01 alone, fully polished (deployed, evaluated, strong README) | quality over quantity beats 3 half-finished projects |

## Lead with these on your resume (showcase ranking)

1. **01 · Financial Research Analyst** — the capstone (multi-agent + CRAG + reflection + metrics + live deploy).
2. **03 · Eval & Observability Platform** — the rarest skill (calibrated judge, injected-regression tests).
3. **15 · Computer-Use / Browser Agent** or **16 · Multimodal** — the demo that makes people stop scrolling (live screencast / boxed-region answer).
4. **09 · MCP Server Trilogy** — a *published* artifact (PyPI + official MCP Registry) anyone can install.
5. **13 · Agent Observability** — production maturity (OTel, cost/latency dashboards).

Put 1–3 above the fold; mention 4–5 as "also built." The rest is interview depth.

## Note on the final gap sweep

Every one of the 146 entries in `500-AI-Agents-Projects` (industry, CrewAI, AutoGen, Agno, LangGraph tables) was checked against the existing set. Three genuine, uncovered patterns were added — **multimodal (16)**, **voice (17)**, **local/offline (18)** — each mapping to an AutoGen or LangGraph row the portfolio didn't touch. Candidates were *declined* where they'd be padding: plan-and-execute / deep-research (covered by 01's orchestration + 15's planner + 03's reflection), the many domain apps (health/trading/tutor/recsys/logistics — applications of already-covered patterns, not new patterns), and niche orchestration variants (AgentBuilder, Society-of-Mind). The portfolio now covers every distinct, hireable 2026 pattern in the source repo.
