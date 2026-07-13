# RESOURCES.md — A2A Multi-Framework Agent Network

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### A2A spec + Python SDK + samples
```bash
git clone --depth 1 https://github.com/a2aproject/A2A.git         /tmp/ref-a2a-spec     # ✅ spec
git clone --depth 1 https://github.com/a2aproject/a2a-python.git  /tmp/ref-a2a-python   # ✅ SDK
pip install a2a-sdk
```
`a2aproject/A2A` is the canonical spec (Linux Foundation project, originally Google); `a2aproject/a2a-python` is the official SDK. Verified in the clone:
- **Discovery path** is `/.well-known/agent-card.json` (spec `docs/topics/agent-discovery.md`, `docs/specification.md`) — **not** `agent.json`.
- **SDK types**: `a2a.types.AgentCard / AgentSkill / AgentCapabilities / Message / Task / Part` (`a2a-python/src/a2a/...`).
- **Runnable examples live in a separate repo**, `a2aproject/a2a-samples` (`samples/python/agents/helloworld`, referenced from `a2a-python/README.md`); the SDK clone also ships `samples/hello_world_agent.py`.
Use these real shapes — the earlier plan's `A2ATaskRequest`/`agent_id` schema was fabricated.

```bash
git clone --depth 1 https://github.com/a2aproject/a2a-samples.git /tmp/ref-a2a-samples   # helloworld + framework agents
```

### CrewAI + LangGraph integration example (read-only)
```bash
git clone --depth 1 https://github.com/crewAIInc/crewAI-examples.git /tmp/ref-crewai-examples
ls /tmp/ref-crewai-examples/integrations/CrewAI-LangGraph   # ✅ (README, main.py, src/)
```
How they bridge a CrewAI crew and a LangGraph graph in one process — informative contrast (your project keeps them as separate A2A services).

### Agno single-agent example (adapt for the Fundamentals specialist)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
ls /tmp/ref-agno/cookbook/02_agents/01_quickstart   # ✅ folder present
```
Cookbook is renumbered `00_quickstart`…`99_docs`; start from `cookbook/02_agents/01_quickstart` for a simple Agno agent, and `cookbook/91_tools/mcp/` for wiring its internal MCP tool.

### MCP Python SDK (each specialist's internal tools)
```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk   # ✅
```

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| A2A spec | `a2aproject/A2A` `docs/` | Read-only — data model + discovery | 0–4 |
| a2a-python SDK + `samples/hello_world_agent.py` | `a2aproject/a2a-python` | Reuse — server/client library | 0–4 |
| a2a-samples helloworld + framework agents | `a2aproject/a2a-samples/samples/python/agents/` | Adapt — the wrap-a-framework-behind-A2A pattern | 1, 2, 3 |
| CrewAI+LangGraph integration | `integrations/CrewAI-LangGraph` | Read-only — in-process bridging contrast | 2 |
| Agno quickstart + MCP | `cookbook/02_agents/01_quickstart`, `cookbook/91_tools/mcp/` | Adapt — Fundamentals specialist | 3 |
| MCP Python SDK | `python-sdk` | Reuse — internal tools | 1, 2, 3 |

## 3. Reused from Project 01
Market-Data (yfinance) and News/Sentiment (search) tool logic lifts nearly as-is from Project 01's workers — the work here is the A2A/MCP wrapping + framework port, not re-solving data fetching.
