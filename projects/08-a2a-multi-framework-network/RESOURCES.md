# RESOURCES.md — A2A Multi-Framework Agent Network

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### Official A2A protocol spec and Python SDK

```bash
git clone --depth 1 https://github.com/a2aproject/A2A.git /tmp/ref-a2a-spec
git clone --depth 1 https://github.com/a2aproject/a2a-python.git /tmp/ref-a2a-python
pip install a2a-sdk
```
Status: ✅ CONFIRMED WORKING. `a2aproject/A2A` is the canonical protocol spec repo (an open-source project under the Linux Foundation, originally contributed by Google); `a2aproject/a2a-python` is the official Python SDK, installable via `pip install a2a-sdk`. Note: the older `google/A2A` and `google-a2a/A2A` URLs also currently resolve (likely redirects/mirrors from before the Linux Foundation transfer) — use `a2aproject/A2A` as the primary reference to avoid depending on a redirect that could be cleaned up later.

### CrewAI + LangGraph Integration example

```bash
git clone --depth 1 https://github.com/crewAIInc/crewAI-examples.git /tmp/ref-crewai-examples
ls /tmp/ref-crewai-examples/integrations/CrewAI-LangGraph
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). Read for how they bridge a CrewAI crew and a LangGraph graph in one process — informative context even though your project keeps them as fully separate services rather than integrated in-process.

### Agno cookbook — agent examples folder

```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
ls /tmp/ref-agno/cookbook/02_agents
```
**⚠️ Partially stale — folder confirmed to exist, specific example file names not verified.** The top-level `cookbook/02_agents` folder was confirmed present during planning (Agno's cookbook is numbered `00_quickstart` through `99_docs`, see Project 01's RESOURCES.md), but individual example filenames within it were not individually checked. Browse it directly for a simple single-agent example to adapt as your Agno-based Fundamentals specialist.

### MCP Python SDK (each specialist's internal tool layer)

```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk
```
Status: ✅ CONFIRMED WORKING (same SDK used in Project 04 and Project 09 — reuse your familiarity from those projects here).

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| A2A protocol spec + Python SDK | Reuse as-is — this is the protocol library every agent and the coordinator build on | Phase 0 through 4 |
| CrewAI+LangGraph integration example | Read-only — informs how you *could* integrate in-process, useful contrast to justify why you're keeping services separate instead | Phase 2 |
| Agno cookbook `02_agents` | Adapt — starting point for your Agno-based Fundamentals specialist | Phase 3 |
| MCP Python SDK | Reuse as-is — internal tool layer for all 3 specialists | Phase 1, 2, 3 |

## 3. Reused from Project 01

The Market-Data (yfinance) and News/Sentiment (search API) tool logic can be lifted nearly as-is from Project 01's corresponding workers — the engineering work in this project is the A2A/MCP wrapping and the framework port, not re-solving the underlying data-fetching logic.
