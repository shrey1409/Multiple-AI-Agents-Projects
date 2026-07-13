# PROFESSOR-NOTES.md — MCP-Native Personal Ops Agent

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| JSON-RPC basics | MCP is JSON-RPC under the hood; helps demystify "what is actually being sent" | Any short JSON-RPC 2.0 spec summary |
| What a "tool" vs. "resource" vs. "prompt" means in MCP | You'll design your servers' interfaces around this exact vocabulary | The official Model Context Protocol specification/docs |
| Basic OAuth 2.0 flow (if using real Calendar/GitHub APIs) | You'll hit this the moment you move off the local stub | Google's or GitHub's own OAuth quickstart |
| Embeddings/semantic search (same as Project 01 prerequisite) | Your memory store needs this for recall | Reuse what you learned in Project 01 |

## 2. Core Concepts Taught

### Model Context Protocol (MCP)
**What:** an open standard (originated by Anthropic, now broadly adopted) for how an AI application connects to external tools and data sources — a common "USB-C port" for agent tool access, instead of every app inventing its own bespoke tool-calling glue.
**Why it exists:** before MCP, every agent framework had its own tool-definition format, so a "GitHub tool" built for one framework couldn't be reused in another. MCP standardizes the server side (what a tool provider exposes) so any compliant client can use any compliant server.
**How it works:** an MCP *server* exposes `tools` (callable actions), `resources` (readable data), and `prompts` (reusable prompt templates) over a JSON-RPC-based protocol (stdio or HTTP+SSE transport); an MCP *client* (your agent, or Claude Desktop, or any other MCP-aware app) discovers and calls them without needing custom integration code per server.
**Where it's used here:** the entire project's premise — you are authoring 3 MCP servers, not just calling "the calendar API" from inline agent code.

### Tool-provider vs. tool-consumer roles
**What:** the distinction between building an agent that *uses* tools (consumer) and building the tool/server that *any* agent could use (provider/author).
**Why it exists:** most agent tutorials teach the consumer side only (call this API from your agent); authoring a well-designed, spec-compliant, reusable server is a different and rarer skill, closer to API design than prompt engineering.
**Where it's used here:** this project deliberately emphasizes the provider side — Project 09 (MCP Server Trilogy) goes even further into pure authoring; this project pairs authoring with being your own first consumer.

### Long-term agent memory
**What:** persisting information across sessions (user preferences, past decisions) so an agent doesn't start from zero every conversation, retrieved via semantic search rather than replaying full transcripts.
**Why it exists:** context windows are finite and expensive; and more importantly, "remembering" a stated preference from last week is a qualitatively different (and much more useful) capability than a stateless chatbot.
**Where it's used here:** the Memory Store, and specifically the recall self-test in PLAN.md §6 — the part that proves the memory actually works, rather than just existing.

### Proactive agents
**What:** an agent that initiates action on a schedule or trigger, rather than only responding to a user's message.
**Why it exists:** "persistent, proactive, memory-backed agents" are called out as a flagship trend precisely because most deployed agents today are reactive chatbots; proactivity is what makes an agent feel like an assistant rather than a search box.
**Where it's used here:** the Scheduler-triggered daily digest and proposed actions.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | MCP Inspector workflow, protocol basics | Directly hireable: "I can author and validate an MCP server" is a specific, checkable claim |
| 1 (Notes server) | Simplest possible MCP server end-to-end | Builds confidence in the protocol mechanics before adding real external-API complexity |
| 2 (GitHub server) | Wrapping a real, rate-limited external API as an MCP tool provider | Nearly identical to what a "build an MCP server for our internal API" task at a real job looks like |
| 3 (Calendar server) | Handling OAuth-backed external services, resource vs. tool design decisions | Same as above, plus the OAuth experience most agent tutorials skip |
| 4 (Agent+Memory) | Building an MCP *client*, semantic memory retrieval | Completes the round-trip: you understand both sides of the protocol, not just one |
| 5 (Proactive+Approval) | Scheduled agent triggers, approval-gated external actions | Reuses and reinforces the HITL pattern from Project 02 in a different, lower-stakes context |
| 6 (Publish) | Packaging and documenting a piece of code for others to install and use | A public, working, documented MCP server on your GitHub is a concrete, checkable artifact — stronger than a screenshot |

## 4. Common Misconceptions & Mistakes

- **Calling a Python function "the MCP server."** If there's no separate process speaking the protocol, and no way for MCP Inspector or another MCP client to connect to it independently of your agent's code, you've built a tool wrapper, not an MCP server.
- **Skipping MCP Inspector validation.** Compliance bugs (wrong schema types, missing required fields) are invisible until a *different* client tries to use your server — Inspector exists specifically to catch this before you find out the hard way.
- **Treating "resources" and "tools" as interchangeable.** They serve different purposes in the spec (resources = readable context/data, tools = callable actions with side effects); picking the wrong one for a given capability is a common design mistake graders will notice.
- **Memory as a growing chat transcript.** This doesn't scale and doesn't generalize across sessions well; structured, embedded, retrievable memory entries are the actual pattern being taught.
- **Under-gating "small" proactive actions.** "It's just a calendar invite, not a big deal" is exactly the reasoning that erodes a HITL system's guarantees over time — keep the rule absolute.

## Understanding-check questions

**After §2 (MCP):** Why does an MCP server being a separate process (rather than a function inside your agent) matter for reusability? What would break if you skipped that separation?

**After §2 (Tool-provider vs. consumer):** Why is authoring a good MCP server a different skill from calling one well? Give an example design decision a server author has to make that a consumer never thinks about.

**After §2 (Long-term memory):** Why is semantic (embedding-based) recall better than keyword search for "did I ever say I don't like Friday afternoon meetings"? What would keyword search miss?

**After §2 (Proactive agents):** What's the risk of a proactive agent that isn't tuned for precision, and how does the false-positive-rate metric in this project's eval strategy address it?

**After Phase 6 (Publish):** What makes an MCP server actually reusable by a stranger — beyond "the code works on my machine"? List at least three things your README/packaging needs to cover.
