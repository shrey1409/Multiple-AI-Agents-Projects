# PROFESSOR-NOTES.md — MCP-Native Personal Ops Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| JSON-RPC 2.0 | MCP is JSON-RPC under the hood | JSON-RPC 2.0 spec summary (it's short) |
| MCP tools vs. resources vs. prompts | You design your servers around this vocabulary | Official MCP spec at modelcontextprotocol.io → "Server Concepts" |
| OAuth 2.0 flow (if real APIs) | You hit it moving off the stub | Google/GitHub OAuth quickstart |
| Embeddings/semantic search | Memory recall needs it | Reuse Project 01's embedding prerequisite |

## 2. Core Concepts Taught

### Model Context Protocol (MCP)
**What.** An open standard for how an AI app connects to tools/data — a common port so every app doesn't invent bespoke tool glue.
**Why it exists.** Before MCP, each framework had its own tool-definition format; a "GitHub tool" for one couldn't be reused in another. MCP standardizes the *server* side so any compliant client uses any compliant server.
**How it works (mechanism + lifecycle).** Client and server first do a **capability-negotiation handshake** (`initialize` → server advertises which of tools/resources/prompts it supports, and protocol version). Then the client can `list_tools`/`call_tool`, `list_resources`/`read_resource`, etc., over JSON-RPC. Transport is **stdio** (local subprocess) or **Streamable HTTP** (remote; the older HTTP+SSE transport is deprecated). Tool schemas are generated from your Python type hints (FastMCP turns a typed function into a JSON Schema the client sees) — which is why typing matters for Inspector validation.
**Where here.** The entire premise — you author 3 servers, not inline tool calls.

### Tool-provider vs. tool-consumer roles
**What.** Building the agent that *uses* tools (consumer) vs. the server *any* agent could use (provider/author).
**Why it exists.** Most tutorials teach the consumer side; authoring a spec-compliant, reusable server is closer to API design (what's a tool vs. resource, what's idempotent, what error shape helps the caller) — a rarer skill.
**Where here.** You do both; Project 09 goes deeper on pure authoring.

### Long-term agent memory
**What.** Persisting info across sessions (preferences, decisions), retrieved by semantic search, not by replaying transcripts.
**Why it exists.** Context windows are finite and expensive; and "remembering a preference from last week" is a qualitatively more useful capability than a stateless chatbot. The hard parts aren't storage — they're the **write policy** (when is something worth remembering?), **consolidation** (dedup near-duplicates), and **supersession** (handle "I changed my mind" without losing history).
**Where here.** The Memory store with write/consolidate/supersede/decay policies (PLAN §2).

### Proactive agents
**What.** An agent that initiates on a schedule/trigger, not only in response to a message.
**Why it exists.** Most deployed agents are reactive chatbots; proactivity is what makes an assistant feel like an assistant. Its risk is precision — an untuned proactive agent is a notification firehose.
**Where here.** The scheduler-triggered daily digest + the FP-rate metric that keeps it honest.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Inspector workflow, protocol basics | "I can author + validate an MCP server" is a checkable claim |
| 1 | Simplest server end-to-end | Confidence in mechanics before external-API complexity |
| 2 | Wrapping a rate-limited real API as a provider | Mirrors "build an MCP server for our internal API" |
| 3 | OAuth-backed service, resource-vs-tool design | The OAuth experience most tutorials skip |
| 4 | Building an MCP *client* + semantic memory | You understand both sides of the protocol |
| 5 | Scheduled triggers + structurally-gated actions | Reinforces HITL in a lower-stakes context |
| 6 | Packaging code a stranger can install | A public working server beats a screenshot |

## 4. Common Misconceptions & Mistakes

- **Calling a Python function "the MCP server."** No separate process, no independent client connection → it's a wrapper.
- **Skipping Inspector.** Compliance bugs are invisible until a *different* client tries your server.
- **Tools/resources interchangeable.** Resources = readable context; tools = actions with effects.
- **Memory as a growing transcript.** Doesn't scale or generalize.
- **Under-gating "small" actions.** Erodes the guarantee over time.

## 5. Understanding-check questions (with answer key)

**Q1 (MCP).** Why does the server being a separate process matter for reusability? What breaks if you skip it?
**A1.** A separate process speaking the protocol can be used by *any* MCP client (Claude Desktop, Inspector, another agent). If it's an in-process function, only your code can call it — you've lost the entire interoperability benefit of building on a standard.

**Q2 (Provider vs. consumer).** Name a design decision a server author faces that a consumer never does.
**A2.** Whether a capability is a `tool` or a `resource`; what error shape to return on bad input; whether an operation is idempotent (e.g., should `complete_task` on an already-done task error or no-op). Consumers just call what exists.

**Q3 (Memory).** Why is semantic recall better than keyword search for "did I say I dislike Friday afternoon meetings"?
**A3.** The stored memory might be phrased "no meetings late in the week" — no keyword overlap with "Friday afternoon," but high semantic similarity. Keyword search misses paraphrases; embedding search retrieves by meaning.

**Q4 (Proactivity).** What's the risk of an untuned proactive agent, and how does the FP metric address it?
**A4.** It nags constantly and gets ignored — indistinguishable from a dumb polling script. Tracking false-positive rate forces you to tune relevance (memory-informed filtering) until notifications are actually worth reading.

**Q5 (Publishing).** Beyond "the code works on my machine," what three things make a server reusable by a stranger?
**A5.** (1) Independent packaging (`pyproject.toml`, `pip install`-able, no hidden deps on your agent); (2) a README with install + config + example tool calls; (3) Inspector-verified schema compliance so a different client can actually connect.
