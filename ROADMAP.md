# ROADMAP — AI Agent Portfolio Build Order

**Fifteen projects.** The original 8 (from `AI-Agent-Project-Ideas.md`), plus 4 that Sonnet added to cover MCP authoring, RAG depth, production safety, and CI/CD (09–12), plus **3 added in the Fable-5 revision** to fill genuine 2026 gaps: **13 Agent Observability**, **14 Long-Term Memory**, **15 Computer-Use/Browser**. Full plans, first-principles teaching notes, and verified resource lists for all 15 live in `projects/<NN-name>/`. The audit of the prior version and the reasoning for the new projects are in `AUDIT.md`.

## Shared contracts (read this first — it's what makes the set cohere)

Two contracts, introduced in the revision, connect the projects. Pin them before building anything that crosses a project boundary:

- **Target Agent Contract.** A compliant agent exposes an HTTP `/invoke` that returns `{output, trajectory, version, cost_usd, latency_ms}`, where `trajectory` is the ordered tool calls with tokens/latency (or an OpenTelemetry GenAI export of the same). **Emitted by** 01, 02, 05, 08, 15. **Consumed by** 03 (eval), 11 (guardrail allow-list), 12 (release gate). **Reference implementation:** Project 13 (OTel). This resolves the black-box-vs-introspection contradiction that ran through Sonnet's 03/11.
- **MCP Server Contract.** stdio locally, **Streamable HTTP** remotely (not the deprecated SSE), Inspector-clean, typed schemas. Used by 04, 08, 09.

## Why this order

Sequenced by **dependencies** (some projects need another's output) and **skill compounding** (each stage reuses the last), not numeric order.

## Stage 1 — Foundations (~weeks 1–4)

**01 · Multi-Agent Financial Research Analyst.** Build first. It teaches the supervisor pattern, agentic/corrective RAG, tool design, and a reflection loop — the skills every later project reuses or references — and it's the first emitter of the Target Agent Contract. Nearly everything downstream points at 01, reuses its specialists, or reuses its ingested data.

## Stage 2 — Prove It Works & See Inside (~weeks 5–8)

**13 · Agent Observability Stack** → **10 · RAG Architecture Bake-Off** → **03 · Agent Evaluation & Observability Platform**

With one agent built, make it *observable* and *proven*. **13 first** (moved early in the revision): instrument Project 01 with OpenTelemetry, which makes the Target Agent Contract real and gives you traces to debug everything that follows. **10** deepens the RAG pattern you used once in 01 (reusing its corpus) into a controlled 4-variant comparison. **03** then builds the eval harness, pointed at 01 as its first Contract-compliant target. You now have an agent, proof it works, and the ability to see why when it doesn't.

*Dependencies:* 13 and 03 need Project 01; 03 consumes the Contract that 13 implements. 10 reuses 01's corpus.

## Stage 3 — Protocols (~weeks 9–12)

**09 · MCP Server Trilogy** → **04 · MCP-Native Personal Ops Agent** → **14 · Long-Term Memory System** → **08 · A2A Multi-Framework Network**

Protocol + statefulness literacy. **09** teaches MCP *authoring* (the rarer half) in isolation and publishes to the official registry. **04** makes you your own first consumer of MCP servers, adding proactive scheduling and a first taste of memory. **14** then makes memory a first-class system and swaps into 04 as its reuse proof. **08** rebuilds 01's specialists across 3 frameworks over A2A — interoperability, not single-framework proficiency.

*Dependencies:* 04 works best after 09 (consume your own servers); 14 plugs into 04; 08 needs 01's specialists to port.

## Stage 4 — Production Patterns (~weeks 13–17)

**02 · Document-Processing HITL Pipeline** → **11 · Agent Guardrail Gateway** → **12 · Agent Release Gate (CI/CD)**

The patterns enterprises actually ship. **02** teaches durable interrupt/resume + idempotency (and emits the Contract). **11** builds a reusable safety gateway that reads the Contract's proposed actions; point it at 01/02. **12** packages 03's harness as an installable GitHub Action, protecting 01 and 02 as its two target repos.

*Dependencies:* 11 needs 01/02 as protected targets + the Contract; 12 needs 03's harness and both 01 and 02.

## Stage 5 — Specialization (optional, ~weeks 18+)

Pick by target role, not all of them:

- **15 · Computer-Use / Browser Agent** — the strongest live demo; pick if you want a showstopper or are targeting computer-use/automation roles.
- **05 · Self-Healing SQL Analytics Agent** — data/analytics-engineering-adjacent; a portable Spider benchmark number.
- **06 · Codebase Onboarding Agent** — dev-tools/platform-engineering; run it live on the interviewer's repo.
- **07 · Red-Team vs Blue-Team Arena** — security-adjacent; strictly scoped to defensive-research/CTF framing (read its scope note first).

## Dependency graph (summary)

```
01 ──┬──> 13 (instruments 01; implements Target Agent Contract)
     ├──> 10 (reuses 01 corpus)
     ├──> 03 (evaluates 01; consumes Contract from 13) ──> 12 (packages 03's harness)
     ├──> 08 (reuses 01's specialists across frameworks)
     └──> 11 (protects 01/02; reads Contract) 
02 ──┬──> 11
     └──> 12 (second target repo)
09 ──> 04 (consumes 09's servers) ──> 14 (memory system plugs into 04)
15 (standalone; emits Contract)   05 · 06 · 07 (standalone specializations)
```

## Timelines

| Track | Scope | Realistic timeline |
|---|---|---|
| **Full mastery** | All 15 | ~7–10 months part-time (10–15 hrs/wk); ~4–5 months full-time |
| **Core production portfolio** | 01 + 13 + 03 + 09 + 02 | ~3–3.5 months part-time — the coherent story: *build → observe → prove → author protocols → ship a durable pattern* |
| **Minimum viable** | 01 + 03 + one of (09 / 08) | ~2–2.5 months part-time — *builds agents → proves they work → makes them interoperable* |
| **Fast-track to interview-ready** | 01 alone, fully polished (deployed, evaluated, strong README) | quality over quantity beats 3 half-finished projects |

## Lead with these on your resume (the showcase ranking)

The spec asked which projects to *showcase first*. Ranked by hiring signal per unit of reviewer attention:

1. **01 · Financial Research Analyst** — the capstone. Multi-agent orchestration + CRAG + reflection + a metrics table + a live deploy. Leads every resume.
2. **03 · Eval & Observability Platform** — the rarest skill. "I can prove an agent works and catch a regression, with calibrated judge agreement (κ) and injected-regression tests." Almost no candidate has this.
3. **15 · Computer-Use / Browser Agent** — the demo that makes people stop scrolling. A live screencast of an agent completing a real web task, with a safety gate.
4. **09 · MCP Server Trilogy** — a *published* artifact (PyPI + official MCP Registry) anyone can `pip install`. Concrete and checkable.
5. **13 · Agent Observability** — signals production maturity (OTel GenAI conventions, cost/latency dashboards) that separates "I built a demo" from "I can operate agents."

Put 1–3 above the fold; mention 4–5 as "also built." The remaining projects are depth you discuss in interviews, not headline bullets.

## A note on the revision

This roadmap replaces the 12-project version. `AUDIT.md` records what the prior plans got right, what was deepened, and why the three new projects were added rather than cut/merged (nothing was cut — 03↔12 and 04↔09 overlaps are intentional layering, sharpened in each plan). Resource paths in every project were re-verified against fresh upstream clones — most notably, the LangGraph tutorials the old plans reported as "404, find them yourself" are simply relocated to `langgraph/examples/` and are now cited exactly. The `500-AI-Agents-Projects` source (its `agents/` folder of 20 runnable examples and `crewai_mcp_course`) was browsed directly to ground the new-project choices in real 2026 hiring signals: observability, stateful/memory agents, and computer-use.
