# RESOURCES.md — Multi-Agent Financial Research Analyst

All paths below were checked on **2026-07-13**. Status legend: ✅ CONFIRMED WORKING (HTTP 200 at time of check) · ⚠️ STALE/RESTRUCTURED (path no longer resolves; instructions given for how to re-locate it).

## 1. What to clone / download

### From `shrey1409/500-AI-Agents-Projects` (your curated source repo)

```bash
# You only need the README's curated links (already extracted into Important-AI-Agents.md)
# and, optionally, the crewai_mcp_course for MCP patterns used later in the portfolio (Project 04/09).
# No agent code from this repo's own agents/ folder is needed for Project 01.
git clone --depth 1 https://github.com/shrey1409/500-AI-Agents-Projects.git /tmp/ref-500-agents
```
Status: ✅ CONFIRMED WORKING (repo exists, README fetched in full during planning).

### LangGraph tutorial notebooks (Multi-Agent Supervisor, Corrective RAG, Reflection)

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
# Then locate the current tutorial paths yourself — see the warning below.
find /tmp/ref-langgraph -iname "*supervisor*"
find /tmp/ref-langgraph -iname "*crag*" -o -iname "*corrective*"
find /tmp/ref-langgraph -iname "*reflection*"
```
**⚠️ STALE/RESTRUCTURED.** The curated list's exact links —
`docs/docs/tutorials/multi_agent/agent_supervisor.ipynb`, `docs/docs/tutorials/rag/langgraph_crag.ipynb`, `docs/docs/tutorials/reflection/reflection.ipynb` — all returned **404** on the `main` branch as of 2026-07-13. LangGraph's docs were reorganized after the curated list was written. This planning session's sandbox also blocks the GitHub contents API and jsdelivr for arbitrary repos, so the *new* paths could not be enumerated from here — **you (the executing model) will have normal network access; do the `find` above, or browse `https://langchain-ai.github.io/langgraph/` directly, and update this section with the working path once located.** Don't waste time guessing single-file raw URLs; clone-and-grep is faster and more reliable than iterating on 404s.

### Agno Financial Reasoning Agent

```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
find /tmp/ref-agno/cookbook -iname "*financ*" -o -iname "*reasoning*"
```
**⚠️ STALE/RESTRUCTURED.** `cookbook/examples/agents/reasoning_finance_agent.py` (the curated link) 404s — Agno's cookbook was restructured into numbered top-level folders (`00_quickstart`, `01_demo`, `02_agents`, `05_agent_os`, `09_evals`, `10_reasoning`, `11_memory`, …, confirmed via directory listing on 2026-07-13). The finance-reasoning example most likely now lives under `02_agents/` or `10_reasoning/` — run the `find` above rather than trusting the old path. Cross-check against `https://docs.agno.com` if `find` turns up nothing.

### CrewAI Stock Analysis Crew (read-only reference, not reused code)

```bash
git clone --depth 1 https://github.com/crewAIInc/crewAI-examples.git /tmp/ref-crewai-examples
ls /tmp/ref-crewai-examples/crews/stock_analysis
```
Status: ✅ CONFIRMED WORKING (`crews/stock_analysis/README.md` returned HTTP 200).

### SEC EDGAR filings (data, not code)

```bash
pip install sec-edgar-downloader
python - <<'PY'
from sec_edgar_downloader import Downloader
dl = Downloader("YourCompanyName", "you@example.com")
dl.get("10-K", "AAPL", limit=1)
dl.get("10-Q", "AAPL", limit=1)
PY
```
Use this to pull the 2–3 real filings per eval ticker referenced in PLAN.md §5/§6.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| LangGraph Multi-Agent Supervisor tutorial (once located) | Adapt — your supervisor has a 4-worker branch + report generator + critic, which the tutorial's simpler 2-worker version doesn't cover; use it for the `Command(goto=...)` routing idiom only | Phase 1, 3 |
| LangGraph Corrective RAG tutorial (once located) | Adapt — reuse the grade-then-fallback control flow; swap in your own SEC-filing retriever and chunking | Phase 0, 2 |
| LangGraph Reflection tutorial (once located) | Adapt — reuse the critique-and-loop-back structure; replace its generic rubric with the financial-report rubric in PLAN.md §6 | Phase 4 |
| Agno Financial Reasoning Agent (once located) | Read-only for understanding — see how they structure Yahoo-Finance tool calls and prompt the model to reason about stock data before answering; don't import Agno itself (this project is LangGraph-native) | Phase 2 (quant/market-data agent design) |
| CrewAI Stock Analysis Crew | Read-only — compare its role decomposition (researcher/analyst/writer) against your worker split; useful for sanity-checking your agent roster, not for code reuse | Phase 2, 3 |
| SEC EDGAR filings | Reuse as raw data for your vector store | Phase 0 |

## 3. Also useful (not in the curated list, found during planning)

- `shrey1409/500-AI-Agents-Projects` → `crewai_mcp_course/` — not needed for this project, but worth skimming later if you continue to Project 04/09 (MCP-focused projects) since it shows MCP server basics in a CrewAI context.
