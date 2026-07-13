# RESOURCES.md — MCP Server Trilogy

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### Official MCP Python SDK, reference servers, Inspector
```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk   # ✅
git clone --depth 1 https://github.com/modelcontextprotocol/servers.git    /tmp/ref-mcp-servers      # ✅
git clone --depth 1 https://github.com/modelcontextprotocol/inspector.git  /tmp/ref-mcp-inspector    # ✅
```
Same repos as Project 04 — reuse your setup. `servers/` has multiple official reference implementations; read 2–3 structurally closest to your API-wrapper / knowledge-store / task-tracker before writing your own. The SDK docs cover **Streamable HTTP** transport (use it for remote; SSE is deprecated).

### Official MCP Registry (publishing target — Sonnet left this vague)
```bash
git clone --depth 1 https://github.com/modelcontextprotocol/registry.git /tmp/ref-mcp-registry   # ✅
# Live registry API: https://registry.modelcontextprotocol.io  (✅ 200)
```
The official registry uses a **`server.json`** manifest. This is the concrete publishing destination for Phase 6 — named now, not "search at implementation time." PyPI (`pypi.org`) is the package host for the installable server.

### crewai_mcp_course (FastMCP walkthrough)
```bash
git clone --depth 1 https://github.com/shrey1409/500-AI-Agents-Projects.git /tmp/ref-500-agents
```
The server-authoring example is **`crewai_mcp_course/lesson_03/mcp_server.py`** ✅ (lesson_01/02 are agent-only). Adapt for a lighter FastMCP server if the official SDK feels heavy for the simplest server.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| MCP Python SDK | `python-sdk` | Reuse — build all 3 | 1, 2, 3 |
| MCP reference servers | `servers/` (2–3) | Read-only — idiomatic structure | 0, 1 |
| MCP Inspector | `inspector` | Reuse — compliance gate | 0–6 |
| Official MCP Registry | `modelcontextprotocol/registry` + live API | Reuse — publishing target (`server.json`) | 6 |
| crewai_mcp_course lesson_03 | `crewai_mcp_course/lesson_03/mcp_server.py` | Adapt — minimal FastMCP server | 1 |

## 3. External resources per server (data/API)
- **Server 1:** a free-tier, low-friction weather/geocoding API — prioritize install ease for whoever clones it.
- **Server 2:** no external API — index this portfolio's own `projects/*/*.md`.
- **Server 3:** no external API — local SQLite.
