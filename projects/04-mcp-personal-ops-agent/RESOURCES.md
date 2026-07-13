# RESOURCES.md — MCP-Native Personal Ops Agent

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### Official MCP Python SDK, reference servers, and Inspector

```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk
git clone --depth 1 https://github.com/modelcontextprotocol/servers.git /tmp/ref-mcp-servers
git clone --depth 1 https://github.com/modelcontextprotocol/inspector.git /tmp/ref-mcp-inspector
```
Status: ✅ CONFIRMED WORKING (all three READMEs returned HTTP 200). These are your primary references, not the curated list — the SDK is what you build servers with, `servers/` has multiple real reference implementations (read a couple before writing your own to see idiomatic tool/resource/prompt definitions), and `inspector/` is the compliance-checking tool used throughout PLAN.md's eval strategy.

### `crewai_mcp_course` from your curated source repo

```bash
git clone --depth 1 https://github.com/shrey1409/500-AI-Agents-Projects.git /tmp/ref-500-agents
ls /tmp/ref-500-agents/crewai_mcp_course
```
Status: ✅ CONFIRMED WORKING (repo verified; course structure was: `lesson_01` CrewAI setup, `lesson_02` MCP server integration, `lesson_03` hierarchical multi-agent + MCP, each with `agent.py`/`requirements.txt`/`.env.example`, confirmed via directory listing during planning). Read `lesson_02` specifically for a beginner-friendly walkthrough of building a custom MCP tool with FastMCP — a lighter-weight alternative to the official SDK if you want less boilerplate for the Notes server in Phase 1.

### Agno MCP Airbnb Agent (MCP-client reference pattern)

```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
find /tmp/ref-agno/cookbook -iname "*airbnb*" -o -iname "*mcp*"
```
**⚠️ STALE/RESTRUCTURED.** `cookbook/examples/agents/airbnb_mcp.py` (curated link) 404s on `main` — same Agno cookbook restructuring noted in Projects 01/02 (now numbered `00_quickstart`...`99_docs`). This example is useful purely as an MCP-*client* reference (how an Agno agent calls an MCP server) — run the `find` above, or check `91_tools/` (tool-integration folder) in the new structure first.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| MCP Python SDK | Reuse as-is — this is the library you build all 3 servers on | Phase 1, 2, 3 |
| MCP reference `servers/` repo | Read-only for pattern reference; pick 1–2 similar servers (e.g. a filesystem or git server) to see idiomatic structure before writing your own | Phase 0, 1 |
| MCP Inspector | Reuse as-is — your compliance-checking tool for every phase | Phase 0 through 5 (validation gate) |
| `crewai_mcp_course` lesson_02 | Adapt — FastMCP patterns for a lighter-weight server if you don't want the official SDK's boilerplate | Phase 1 |
| Agno MCP Airbnb Agent (once located) | Read-only — MCP-client calling pattern reference only; you're building your own client in Phase 4, not reusing Agno | Phase 4 |

## 3. External API docs (not repos, but load-bearing)

- Google Calendar API docs (for the real backend option in Phase 3) — only needed once you move off the local `.ics` stub.
- GitHub REST API docs (for the GitHub MCP server in Phase 2) — a fine-grained personal access token with `repo` (read) and pull-request comment scopes is sufficient; don't request broader scopes than needed.
