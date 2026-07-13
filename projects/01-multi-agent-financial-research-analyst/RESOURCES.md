# RESOURCES.md — Multi-Agent Financial Research Analyst

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed present at the exact path given. Where Sonnet's original file said "path 404s, find it yourself," the **actual current path** is given below — no deferral to implementation time.

## 1. What to clone / download

### LangGraph tutorial notebooks (Supervisor, Corrective RAG, Reflection)
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
The LangGraph docs were reorganized: tutorials moved from `docs/docs/tutorials/` to **`examples/`**. Exact current paths (✅ all present):
- Corrective RAG: `examples/rag/langgraph_crag.ipynb` (and `langgraph_crag_local.ipynb`)
- Reflection: `examples/reflection/reflection.ipynb`
- Supervisor/multi-agent routing: `examples/multi_agent/multi-agent-collaboration.ipynb` (there is **no** standalone `agent_supervisor.ipynb`; the prebuilt supervisor also lives in the separate `langgraph-supervisor` PyPI package)

### CrewAI Stock Analysis Crew (read-only design reference)
```bash
git clone --depth 1 https://github.com/crewAIInc/crewAI-examples.git /tmp/ref-crewai-examples
ls /tmp/ref-crewai-examples/crews/stock_analysis   # ✅ present
```

### Agno finance/reasoning reference (read-only)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
```
⚠️ The old `cookbook/examples/agents/reasoning_finance_agent.py` is **deleted, not moved** — the cookbook was renumbered `00_quickstart`…`99_docs`. For yfinance-style tool usage read examples under `cookbook/02_agents/04_tools/` and reasoning under `cookbook/10_reasoning/`. Treat as pattern reference only; this project is LangGraph-native.

### SEC EDGAR filings (data)
```bash
pip install sec-edgar-downloader
python - <<'PY'
from sec_edgar_downloader import Downloader
dl = Downloader("YourName", "you@example.com")
dl.get("10-K", "AAPL", limit=1); dl.get("10-Q", "AAPL", limit=1)
PY
```

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse / Adapt / Read-only | Feeds phase |
|---|---|---|---|
| LangGraph CRAG | `examples/rag/langgraph_crag.ipynb` | Adapt — grade-then-fallback control flow; swap your SEC retriever + binary grader | 0, 2 |
| LangGraph Reflection | `examples/reflection/reflection.ipynb` | Adapt — critique-and-loop structure; replace its rubric with §6's | 4 |
| LangGraph multi-agent | `examples/multi_agent/multi-agent-collaboration.ipynb` | Adapt — `Command(goto=...)` routing idiom for your 4-worker branch | 1, 3 |
| CrewAI stock_analysis | `crews/stock_analysis` | Read-only — compare role decomposition to your worker split | 2, 3 |
| Agno tools/reasoning | `cookbook/02_agents/04_tools/`, `cookbook/10_reasoning/` | Read-only — Yahoo-Finance tool + reasoning prompt patterns | 2 |
| SEC EDGAR filings | (data) | Reuse as raw corpus | 0 |

## 3. Also useful
- `explodinggradients/ragas` (`pip install ragas`) — faithfulness/context-precision metrics for the CRAG sub-component; check current metric-import names against the installed version.
- `shrey1409/500-AI-Agents-Projects` → `agents/11-stock-research-agent/` — a single-file reference for a simpler stock agent; skim for tool ideas, not architecture.
