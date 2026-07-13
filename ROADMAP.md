# ROADMAP — AI Agent Portfolio Build Order

Twelve projects: the original 8 from `AI-Agent-Project-Ideas.md`, plus 4 new ones (09–12) proposed to cover 2026's highest-value hiring signals — MCP authoring, RAG-architecture depth, production safety, and CI/CD for agents — that the original 8 only touched partially. Full plans, professor-style teaching notes, and resource lists for all 12 live in `projects/<NN-name>/`.

## Why this order

The sequencing below is designed around **dependencies** (some projects need another project's output to exist first) and **skill compounding** (each stage reuses what the previous one taught, so you're never starting cold). It is not simply the numbering order.

## Stage 1 — Foundations (~weeks 1–4)

**01 · Multi-Agent Financial Research Analyst**

Build this first. It teaches the supervisor pattern, agentic/corrective RAG, tool design, and a reflection loop — the single set of skills every other project in this portfolio reuses or references. Nearly every later project either points at Project 01's agent, reuses its specialists, or reuses its ingested data.

## Stage 2 — Prove It Works (~weeks 5–7)

**10 · RAG Architecture Bake-Off** → **03 · Agent Evaluation & Observability Platform**

With one working agent in hand, go deep on the two things the market says most differentiate candidates: RAG mastery and evaluation rigor. Project 10 deepens the RAG pattern you only used once in Project 01, benchmarking all four LangGraph RAG variants head-to-head (and can reuse Project 01's ingested filings as its corpus). Project 03 then builds the eval harness pattern, pointed at Project 01 as its first real target — you now have both an agent and proof it works, with numbers.

*Dependency:* Project 03 needs a target agent to evaluate; Project 01 must exist first. Project 10 doesn't strictly require Project 01 but reusing its corpus saves real setup time.

## Stage 3 — Protocols (~weeks 8–10)

**09 · MCP Server Trilogy** → **04 · MCP-Native Personal Ops Agent** → **08 · A2A Multi-Framework Agent Network**

Now build protocol literacy. Project 09 teaches MCP *authoring* (the rarer, more hireable half of MCP skill) in isolation from any one agent's business logic. Project 04 then makes you your own first consumer of those servers, adding memory and proactive scheduling. Project 08 reuses Project 01's specialists, rebuilt across 3 frameworks and connected over A2A — proving interoperability, not just single-framework proficiency.

*Dependency:* Project 04 works best after Project 09 (you consume your own servers) but can be reordered to consume a public MCP server first if you want MCP concepts sooner. Project 08 needs Project 01's specialists to exist to port.

## Stage 4 — Production Patterns (~weeks 11–14)

**02 · Document-Processing HITL Pipeline** → **11 · Agent Guardrail Gateway** → **12 · Agent Release Gate (CI/CD)**

This stage is about the patterns enterprises actually ship: durable human-in-the-loop approval, a reusable safety layer, and eval-as-CI. Project 02 teaches interrupt/resume durable execution and idempotency. Project 11 builds a standalone guardrail gateway and needs an existing agent to protect — point it at Project 01 and/or 02. Project 12 packages Project 03's eval harness as an installable GitHub Action and needs that harness (and ideally two target repos — Project 01 and 02) to prove it's genuinely reusable.

*Dependency:* Project 11 needs Project 01/02 to exist as protected targets. Project 12 needs Project 03's harness and benefits from having both Project 01 and 02 available as the two target repos in its reusability proof.

## Stage 5 — Specialization (optional, ~weeks 15+)

Pick based on the role you're targeting, not all three:

- **05 · Self-Healing SQL Analytics Agent** — if you're leaning data/analytics-engineering-adjacent roles. Benchmarked on Spider, a genuinely portable "one number on your resume" result.
- **06 · Codebase Onboarding Agent** — if you're leaning dev-tools/platform-engineering roles. The most instantly demoable project in the portfolio (run it live on an interviewer's repo).
- **07 · Red-Team vs Blue-Team Agent Arena** — if you're leaning security-adjacent roles. Strictly scoped to a defensive-research/CTF-lab framing; read its PLAN.md's scope note before starting.

## Dependency graph (summary)

```
01 ──────────────┬─────────────┬──────────────┐
                  │             │              │
                  v             v              v
                 10            03             08 (reuses 01's specialists)
                                │
                                v
                               12 (packages 03's harness)
                                ^
                                │
02 ───┬──────────────────────────
      v
     11 (protects 01 and/or 02)

09 ──> 04 (consumes 09's servers)
```

## Timelines

| Track | Scope | Realistic timeline |
|---|---|---|
| **Full mastery track** | All 12 projects | ~6–9 months part-time (10–15 hrs/week); ~3–4 months if built full-time |
| **Minimum viable portfolio** | 1 capstone + 2 mid-size, per the original brief's own recommendation | 01 + 03 + one of (09 or 08) → ~2–2.5 months part-time. This sequence tells the coherent story the original brief calls out: *builds agents → proves they work → makes them interoperable.* |
| **Fast-track to interview-ready** | If you need something to show *this month* | 01 alone, fully polished (deployed, evaluated informally, well-written README) beats 3 half-finished projects — quality over quantity applies even more under time pressure |

## A note on the new projects (09–12)

These were proposed during planning after browsing `shrey1409/500-AI-Agents-Projects`'s current structure (its `agents/` folder of 20 runnable examples and its `crewai_mcp_course`) and cross-referencing 2026 hiring signals: MCP server *authoring* (09) is rarer than MCP *consumption* (04); a controlled RAG bake-off (10) turns "I know 4 RAG patterns" from a claim into a data-backed comparison; a standalone guardrail gateway (11) is the production-safety-as-a-product angle the original 8 don't cover; and packaging eval-as-CI (12) turns Project 03's harness into an installable tool, not just a personal script. Each project folder's own PLAN.md §"Why this project exists" repeats this justification in context.
