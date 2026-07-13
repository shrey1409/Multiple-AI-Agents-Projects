# RESOURCES.md — Multi-Agent Financial Research Analyst (India / NSE-BSE)

Framework references verified 2026-07-13 against fresh clones. India data libraries verified on PyPI 2026 (versions noted). ✅ = confirmed. The framework/pattern references are **unchanged** from the original — only the data layer is localized.

## 1. Framework & pattern references (unchanged, market-neutral)

### LangGraph tutorial notebooks (Supervisor, Corrective RAG, Reflection)
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Tutorials moved from `docs/docs/tutorials/` to **`examples/`** (✅ all present):
- Corrective RAG: `examples/rag/langgraph_crag.ipynb` (and `langgraph_crag_local.ipynb`)
- Reflection: `examples/reflection/reflection.ipynb`
- Supervisor/multi-agent routing: `examples/multi_agent/multi-agent-collaboration.ipynb` (no standalone `agent_supervisor.ipynb`; prebuilt supervisor is the `langgraph-supervisor` PyPI package)

### CrewAI Stock Analysis Crew (read-only design reference)
```bash
git clone --depth 1 https://github.com/crewAIInc/crewAI-examples.git /tmp/ref-crewai-examples
ls /tmp/ref-crewai-examples/crews/stock_analysis   # ✅ present (US-oriented; compare role split, not data)
```

### Agno finance/reasoning reference (read-only)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
```
⚠️ Old `cookbook/examples/agents/reasoning_finance_agent.py` is deleted (cookbook renumbered `00_quickstart`…`99_docs`). yfinance-style tool usage: `cookbook/02_agents/04_tools/`; reasoning: `cookbook/10_reasoning/`. Pattern reference only.

### RAGAS + the 500-repo stock agent
- `explodinggradients/ragas` (`pip install ragas`) — CRAG-subcomponent metrics; check current metric-import names.
- `shrey1409/500-AI-Agents-Projects` → `agents/11-stock-research-agent/` — simpler single-file stock agent; skim for tool ideas.

## 2. Data & API layer — evaluated options (India)

All library existence verified on PyPI (2026): `yfinance`, `nsepython 2.97`, `jugaad-data 0.33.1`, `bsedata 0.6.0`, `kiteconnect`, `upstox-python-sdk`, `nsepy`. Portal URLs (nseindia.com, bseindia.com, sebi.gov.in, screener.in) are the well-known official/retail sites; the sandbox proxy blocked live fetches, so program against the wrapper libraries (which handle NSE's cookie/header handshake) rather than the raw endpoints.

| Source | What it gives | Cost | Rate limit | Auth | Verdict |
|---|---|---|---|---|---|
| **yfinance** (`.NS`/`.BO`, `^NSEI`,`^BSESN`,`^NSEBANK`) | OHLCV, market cap, basic ratios, indices | Free | Soft/unofficial; back off on empties | None | **DEFAULT MarketDataAdapter** |
| **nsepython** | NSE quotes, price bands (circuit), option chain, corporate announcements, shareholding | Free | Unofficial; be gentle | None | **Default gap-filler** (circuit band, announcements) |
| **jugaad-data** | NSE/BSE historical, **bhavcopy**, F&O, index data | Free | Unofficial | None | Default for bhavcopy / historical / F&O lot data |
| **bsedata** | BSE quotes by scrip code | Free | Unofficial | None | BSE-side quotes when `.BO` needed |
| **Screener.in** | Fundamentals (P/E, ROCE, ROE), promoter holding, concall & annual-report links | Free to browse; **no official public API** | N/A (scrape/export politely) | Login for export | Retail cross-check + a discovery path to filing PDFs |
| **Moneycontrol / Economic Times** | Equity news headlines | Free | RSS + scrape | None | **DEFAULT NewsAdapter** source (+ Tavily fallback) |
| **Tavily** | LLM-oriented web/news search | Free ~1,000 req/mo | Per plan | API key | NewsAdapter fallback (also the US adapter's primary) |
| **Zerodha Kite Connect** | Real-time quotes, L2 depth, historical | **₹2,000/mo per app** (historical add-on separate); needs a Zerodha account | Documented per-endpoint | OAuth (request_token → access_token) | Optional real-time adapter; overkill for a research brief |
| **Upstox API** | Real-time + historical | API free; needs an Upstox account | Documented | OAuth | Optional real-time adapter, cheaper than Kite |
| **SEBI (sebi.gov.in)** | Regulations, circulars, FPI/FII data, SAST/PIT disclosures | Free | N/A | None | Domain/regulatory context + FII flow data |

**Default stack decision:** yfinance(`.NS`/`.BO`) for market data, nsepython/jugaad-data/bsedata for the fields yfinance lacks (circuit bands, shareholding, bhavcopy, F&O), Moneycontrol/ET + Tavily for news. Free, no broker account, no OAuth — right for a research brief. Kite/Upstox are documented adapters for when you need live intraday depth (you don't, for fundamentals research).

### Market-data fetch (default adapter)
```bash
pip install yfinance nsepython jugaad-data bsedata
python - <<'PY'
import yfinance as yf
t = yf.Ticker("RELIANCE.NS")            # NSE; use "500325.BO" for BSE
print(t.fast_info["last_price"], t.fast_info.get("market_cap"))
print(yf.Ticker("^NSEI").history(period="1mo").tail())   # NIFTY 50 index
PY
# circuit band / shareholding / announcements via nsepython (handles NSE cookies):
python - <<'PY'
from nsepython import nse_eq
d = nse_eq("RELIANCE")
print(d["priceInfo"].get("pPriceBand"))   # price band / circuit context
PY
```

### Fundamentals RAG corpus — exactly where it comes from and how to fetch
The corpus is **Indian annual reports, quarterly results, and corporate announcements** (PDFs). Fetch order:
1. **Annual reports** — from the company's investor-relations page (most large-caps host `Annual Report FY20xx.pdf` directly), or **BSE**: bseindia.com → *Corporates → Annual Reports* (by scrip code), or **NSE**: nseindia.com → *Companies → Filings → Annual Reports*.
2. **Quarterly results** (SEBI LODR Reg 33) — NSE/BSE *Financial Results* filing feeds; `nsepython` exposes the announcements/results feed programmatically.
3. **Corporate announcements** (LODR Reg 30) and **shareholding pattern** (Reg 31, quarterly — promoter holding & **pledged %**) — NSE/BSE announcement feeds via `nsepython`; these are the promoter-pledge and material-event sources.
4. **Investor presentations / concall transcripts** — linked from Screener.in's company page (a fast discovery route to the exchange PDFs).

Ingest the PDFs into Chroma with **section-aware chunking on Indian AR structure** (MD&A / Directors' Report / Corporate Governance / BRSR / Notes / Auditor's Report), per PLAN §8. Key the collection on `ticker + filing_period`.

### Worldwide mode (US adapter — kept reachable for comparison)
```bash
pip install sec-edgar-downloader
python - <<'PY'
from sec_edgar_downloader import Downloader
dl = Downloader("YourName", "you@example.com")
dl.get("10-K", "AAPL", limit=1); dl.get("10-Q", "AAPL", limit=1)
PY
```
Selected by `MARKET=US`; the SEC `FilingsAdapter` + section-aware chunking on 10-K Items is the original curriculum, preserved.

## 3. Mapping: reference → project part

| Reference | Verified path/lib | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| LangGraph CRAG | `examples/rag/langgraph_crag.ipynb` | Adapt — grade-then-fallback; swap in the Indian-filings retriever | 0, 2 |
| LangGraph Reflection | `examples/reflection/reflection.ipynb` | Adapt — critique loop; §6 rubric | 4 |
| LangGraph multi-agent | `examples/multi_agent/multi-agent-collaboration.ipynb` | Adapt — routing idiom | 1, 3 |
| yfinance `.NS`/`.BO` | PyPI `yfinance` | Reuse — default MarketDataAdapter | 1, 2 |
| nsepython / jugaad-data / bsedata | PyPI (2.97 / 0.33.1 / 0.6.0) | Reuse — circuit bands, shareholding, bhavcopy, F&O | 2 |
| Moneycontrol/ET + Tavily | RSS/scrape + PyPI `tavily-python` | Reuse — default NewsAdapter + fallback | 2 |
| NSE/BSE filing portals | nseindia.com / bseindia.com (via nsepython) | Reuse — RAG corpus (annual reports/results/announcements) | 0 |
| SEC EDGAR | PyPI `sec-edgar-downloader` | Read-only — worldwide-mode US adapter | 0 (US) |

## 4. Domain references (India — for the agent's knowledge, free)
- **Zerodha Varsity** — fundamental analysis, Indian market structure, F&O basics (the standard free India retail curriculum).
- **SEBI LODR Regulations** (30 material events, 31 shareholding, 33 results) — what each filing type contains, so your chunker and prompts use the right vocabulary.
- **NSE/BSE circular archives** — circuit-band/price-band rules and F&O lot-size master (lot sizes change; fetch the current master via jugaad-data rather than hardcoding).
