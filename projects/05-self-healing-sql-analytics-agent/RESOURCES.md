# RESOURCES.md — Self-Healing SQL Analytics Agent

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### Spider benchmark (data + eval methodology)
```bash
git clone --depth 1 https://github.com/taoyds/spider.git /tmp/ref-spider   # ✅ on the `master` branch (`main` 404s)
ls /tmp/ref-spider   # evaluation.py, process_sql.py, baselines/, evaluation_examples/  ✅
```
Use its bundled SQLite databases + gold-query files (Phase 0/4) and its **`evaluation.py`** for execution-match scoring so your number is comparable to published Spider results. Caveat: this code is Python-2-era — run it under a compatible interpreter or a maintained fork; don't hand-roll comparison logic.

### LangGraph SQL agent tutorial
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Moved to `examples/` — verified path: **`examples/tutorials/sql-agent.ipynb`** ✅. Adapt its schema-introspection-then-generate flow; you add the static guard + self-check/retry loop it doesn't cover.

### AutoGen SQL + self-correction notebooks (read-only, legacy)
```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_sql_spider.ipynb                      # ✅ 200
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_auto_feedback_from_code_execution.ipynb # ✅ 200
```
⚠️ `0.2` branch = legacy (AutoGen in maintenance mode). The Spider notebook shows AutoGen's schema-context/prompt design for this exact benchmark; the auto-feedback notebook is the canonical "execute → check → feed error back → retry" reference for your self-check loop.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| Spider data + `evaluation.py` | `taoyds/spider` (master) | Reuse — dataset + scoring | 0, 4 |
| LangGraph SQL agent | `examples/tutorials/sql-agent.ipynb` | Adapt — introspect-then-generate | 1 |
| AutoGen SQL Spider (0.2) | `notebook/agentchat_sql_spider.ipynb` | Read-only — prompt/schema design | 1, 4 |
| AutoGen auto-feedback (0.2) | `notebook/agentchat_auto_feedback_from_code_execution.ipynb` | Adapt — execute/check/retry control flow | 2 |

## 3. Also useful
- `sqlglot` (`pip install sqlglot`) — parse generated SQL into an AST for the static statement-type guard (Phase 0) and for nicer, structured error feedback than a raw driver exception.
- `shrey1409/500-AI-Agents-Projects` → `agents/04-sql-query-agent/` — a simple single-file NL-to-SQL agent; skim for prompt ideas, not architecture.

## 3. India-specific demo data (localization)
Benchmark stays on **Spider** (unchanged). For the India demo DB, load **NSE bhavcopy** (free, no auth) via `jugaad-data` (PyPI `jugaad-data 0.33.1`, verified):
```bash
pip install jugaad-data
python - <<'PY'
from jugaad_data.nse import bhavcopy_save
from datetime import date
bhavcopy_save(date(2024,1,15), "./")   # daily NSE OHLCV CSV -> load into SQLite/Postgres
PY
```
Then ask the agent "top 10 gainers in the auto sector last month", "avg delivery % for NIFTY 50" etc. INR/lakh-crore in the narrative layer. (An Indian e-commerce/retail open dataset from data.gov.in or Kaggle is an alternative if you prefer non-market data.)
