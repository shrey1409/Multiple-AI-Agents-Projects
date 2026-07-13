# RESOURCES.md — MCP-Native Personal Ops Agent

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed present.

## 1. What to clone / download

### Official MCP Python SDK, reference servers, Inspector
```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk   # ✅
git clone --depth 1 https://github.com/modelcontextprotocol/servers.git    /tmp/ref-mcp-servers      # ✅
git clone --depth 1 https://github.com/modelcontextprotocol/inspector.git  /tmp/ref-mcp-inspector    # ✅
```
Primary references: the SDK builds your servers (FastMCP is in `python-sdk`); `servers/` has real reference implementations — read 1–2 structurally similar to yours before writing; `inspector/` is the compliance gate used throughout §6. Note the SDK's transport docs cover **Streamable HTTP** (use it for remote), not the deprecated SSE transport.

### crewai_mcp_course (FastMCP walkthrough)
```bash
git clone --depth 1 https://github.com/shrey1409/500-AI-Agents-Projects.git /tmp/ref-500-agents
ls /tmp/ref-500-agents/crewai_mcp_course   # lesson_01 lesson_02 lesson_03  ✅
```
**Correction to the original file:** the server-authoring example is in **`lesson_03/mcp_server.py`** (lesson_01/02 are `agent.py`+`requirements.txt` only). Read `lesson_03` for a minimal FastMCP server.

### Agno MCP client reference (read-only)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
```
⚠️ The old `cookbook/examples/agents/airbnb_mcp.py` is **deleted**. Real MCP examples now live under **`cookbook/91_tools/mcp/`** (e.g. `notion_mcp_agent.py`, `brave.py`, `sse_transport/`, `pipedream_google_calendar.py`) and `cookbook/91_tools/mcp_tools.py`. Read these for how an Agno agent *consumes* an MCP server — client-pattern reference only.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| MCP Python SDK (FastMCP) | `python-sdk` | Reuse — build all 3 servers | 1, 2, 3 |
| MCP reference servers | `servers/` (pick 1–2) | Read-only — idiomatic structure | 0, 1 |
| MCP Inspector | `inspector` | Reuse — compliance gate | 0–5 |
| crewai_mcp_course lesson_03 | `crewai_mcp_course/lesson_03/mcp_server.py` | Adapt — minimal FastMCP server | 1 |
| Agno MCP client examples | `cookbook/91_tools/mcp/` | Read-only — client calling pattern | 4 |

## 3. External API docs
- Google Calendar API (for the real backend in Phase 3) — only after the `.ics` stub works.
- GitHub REST API — a fine-grained PAT with repo-read + PR-comment scope is sufficient; request nothing broader.
