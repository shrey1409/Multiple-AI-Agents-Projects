# RESOURCES.md — Agent Evaluation & Observability Platform

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### LangGraph Chatbot Simulation Evaluation tutorial

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
find /tmp/ref-langgraph -iname "*simulation*" -o -iname "*simulat*"
```
**⚠️ STALE/RESTRUCTURED.** The curated link (`docs/docs/tutorials/chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb`) 404s on `main` as of 2026-07-13 — same LangGraph docs reorganization flagged in Project 01/02's RESOURCES.md. This is the single most directly relevant reference in the whole portfolio for this project — worth the extra effort of `find`-ing its current location (or checking LangGraph's live docs site) rather than skipping it.

### AutoGen AgentEval and AgentOps notebooks

```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agenteval_cq_math.ipynb
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_agentops.ipynb
```
Status: ✅ CONFIRMED WORKING (both returned HTTP 200). AgentEval shows a multi-agent-critic approach to grading a target system's outputs — a useful alternative design to a single judge LLM if you want to experiment with a "panel of judges." AgentOps shows tracing/observability instrumentation patterns for tool calls, cost, and errors — directly relevant to the Trace Collector.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| LangGraph Chatbot Simulation Evaluation (once located) | Adapt — reuse the two-agent (simulator ↔ target) loop structure; you extend it with adversarial personas and a separate judge pass, which the tutorial doesn't cover in depth | Phase 1, 2 |
| AutoGen AgentEval notebook | Read-only, optional adaptation — study the "criteria generation then quantify" approach as an alternative judge design if a single-LLM judge feels under-calibrated | Phase 3 |
| AutoGen AgentOps notebook | Read-only — study what fields they instrument (tokens, tool errors, latency) to make sure your Trace Collector schema doesn't miss anything standard | Phase 0, 1 |

## 3. Also useful (not agent code, but load-bearing for this project)

- **RAGAS** (`pip install ragas`) — if your target agent (Project 01) exposes intermediate retrieved context, RAGAS gives you faithfulness/answer-relevancy/context-precision metrics for the RAG sub-component for free, rather than hand-rolling those specific metrics in your judge rubric. Check RAGAS's own docs/GitHub for current API — it has changed versions since any older tutorial you might find.
- **OWASP LLM Top 10** (owasp.org) — read the prompt-injection entry before writing your adversarial persona scenarios in Phase 2, so the injection attempts you simulate reflect real known patterns rather than invented ones.
