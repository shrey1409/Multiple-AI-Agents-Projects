# RESOURCES.md — MCP Server Trilogy

Checked 2026-07-13. ✅ CONFIRMED WORKING.

## 1. What to clone / download

### Official MCP Python SDK, reference servers, and Inspector

```bash
git clone --depth 1 https://github.com/modelcontextprotocol/python-sdk.git /tmp/ref-mcp-python-sdk
git clone --depth 1 https://github.com/modelcontextprotocol/servers.git /tmp/ref-mcp-servers
git clone --depth 1 https://github.com/modelcontextprotocol/inspector.git /tmp/ref-mcp-inspector
```
Status: ✅ CONFIRMED WORKING (all three verified during planning; same repos used in Project 04 — reuse your setup from there if you built it first). `servers/` contains multiple official reference implementations spanning different tool/resource/prompt patterns — read at least 2-3 of them (pick ones structurally closest to your API-wrapper, knowledge-store, and task-tracker designs) before writing your own from scratch.

### `crewai_mcp_course` (FastMCP patterns, lighter-weight alternative)

```bash
git clone --depth 1 https://github.com/shrey1409/500-AI-Agents-Projects.git /tmp/ref-500-agents
ls /tmp/ref-500-agents/crewai_mcp_course
```
Status: ✅ CONFIRMED WORKING (verified during planning for Project 04; same course applies here). `lesson_02` is the relevant one — building a custom MCP tool with FastMCP.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| MCP Python SDK | Reuse as-is — the library all 3 servers are built on | Phase 1, 2, 3 |
| MCP reference `servers/` repo | Read-only pattern reference — study 2-3 for idiomatic tool/resource/prompt structuring | Phase 0, 1 |
| MCP Inspector | Reuse as-is — your compliance-checking tool, every phase | Phase 0 through 6 |
| `crewai_mcp_course` lesson_02 | Adapt — FastMCP as a lighter alternative if the official SDK feels heavy for the simplest server (likely the API-Wrapper) | Phase 1 |

## 3. External resources for each server (data/API, not agent code)

- **Server 1 (API Wrapper):** pick a free-tier, no-friction-signup weather or geocoding API (many exist with generous free tiers and simple API-key or even keyless access) — prioritize ease of setup for whoever installs your server later over any specific provider.
- **Server 2 (Knowledge Store):** no external API needed — index this very portfolio's own markdown files (`projects/*/*.md`) as a ready-made, realistic document corpus.
- **Server 3 (Task Tracker):** no external API — purely local SQLite.

## 4. Where to publish

- **PyPI** (pypi.org) — standard, free, well-documented publishing flow for at least one server.
- A community MCP server directory/registry (check the current state of the ecosystem at implementation time — this space was actively growing as of early 2026 and specific directory names/URLs may have changed; search for "MCP server registry" or "MCP server directory" rather than relying on any single hardcoded URL here, since this is exactly the kind of fast-moving link that risks going stale between planning and execution).
