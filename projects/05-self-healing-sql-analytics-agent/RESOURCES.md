# RESOURCES.md — Self-Healing SQL Analytics Agent

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### Spider benchmark (data + eval methodology)

```bash
git clone --depth 1 https://github.com/taoyds/spider.git /tmp/ref-spider
```
Status: ✅ CONFIRMED WORKING on the **`master`** branch specifically (`main` 404s — this repo predates the `main`-as-default convention, don't assume `main` universally). Use its bundled databases and gold-query files for Phase 0/4; use its evaluation script/methodology (not a hand-rolled comparison) so your reported accuracy is comparable to published Spider results, per PLAN.md §7.

### LangGraph SQL Agent tutorial

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
find /tmp/ref-langgraph -iname "*sql*"
```
**⚠️ STALE/RESTRUCTURED.** The curated link (`docs/docs/tutorials/sql-agent.ipynb`) 404s on `main` as of 2026-07-13 — same LangGraph docs reorganization flagged across this portfolio's other RESOURCES.md files. Run the `find` above to locate the current path; it's the closest direct reference for the schema-inspection + query-generation flow.

### AutoGen NL-to-SQL Spider notebook and auto-feedback code-execution notebook

```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_sql_spider.ipynb
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_auto_feedback_from_code_execution.ipynb
```
Status: ✅ CONFIRMED WORKING (`agentchat_sql_spider.ipynb` returned HTTP 200 during planning; the auto-feedback notebook is the same family of AutoGen 0.2 notebooks already verified working for other projects in this portfolio — spot-check it yourself before relying on it, same URL pattern). The Spider notebook shows AutoGen's own approach to this exact benchmark — read it for their prompt/schema-context design even though you're building in LangGraph, not AutoGen. The auto-feedback notebook is the canonical "self-healing code execution loop" reference for the retry/self-check pattern.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| Spider benchmark data + eval script | Reuse as-is — this is your eval dataset and scoring methodology, don't reinvent it | Phase 0, 4 |
| LangGraph SQL Agent tutorial (once located) | Adapt — reuse the schema-introspection-then-generate flow; you add the self-check/retry loop it likely doesn't cover in depth | Phase 1 |
| AutoGen NL-to-SQL Spider notebook | Read-only — compare their schema-context and prompt design against yours | Phase 1, 4 |
| AutoGen auto-feedback code-execution notebook | Adapt — reuse the "execute, check, feed error back, retry" control-flow idea, applied to SQL instead of general code | Phase 2 |
