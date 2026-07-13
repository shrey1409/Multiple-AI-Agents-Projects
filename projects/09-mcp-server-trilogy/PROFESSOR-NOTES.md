# PROFESSOR-NOTES.md — MCP Server Trilogy

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| MCP fundamentals (tools/resources/prompts, transports) | The whole project is authoring servers | Project 04 first, or the official MCP spec (modelcontextprotocol.io) |
| Package publishing (`pyproject.toml`, PyPI) | Phase 5/6 actually publish | PyPI "Packaging Python Projects" tutorial |
| Testing a subprocess over a real transport | Your tests must exercise the protocol, not internals | Python `subprocess` docs; pytest fixtures for external processes |
| JSON Schema (generated from type hints) | Inspector validates your tools' schemas | Note that FastMCP derives the schema from your Python type annotations |

## 2. Core Concepts Taught

### Tool-provider authorship as a distinct skill
**What.** Designing/building the *server* side — what to expose, how to type it, how to fail — vs. consuming someone's server.
**Why it exists.** Most "I built an agent" projects only exercise the consumer side. Authoring forces API-design judgment (tool vs. resource, what's idempotent, what error shape helps a caller) that consuming never does.
**Where here.** All 3 servers.

### Protocol compliance vs. "it works for me"
**What.** The gap between "works when *my* client calls it" and "verifiably correct against the spec, usable by *any* client."
**Why it exists.** A server coupled to your own client's assumptions isn't reusable — the whole value of a standard protocol is that anyone's compliant client works, and that claim is only as strong as your verification.
**How it works (mechanism).** The MCP handshake negotiates capabilities and protocol version; the client then introspects your tools via their generated JSON Schemas. A different client will send exactly-spec-shaped requests — if your server assumed something extra your own client always sent, it breaks. Inspector and a second client surface that.
**Where here.** The Phase-4 cross-client verification — the decisive validation.

### Idempotency and error semantics as API design
**What.** Deciding, deliberately, what happens on a repeated call or a bad input — and making it predictable and typed.
**Why it exists.** A provider that crashes on bad input or gives different results on retries is unsafe to build agents against. This is API design 101 in the MCP context.
**Where here.** `complete_task`'s idempotent completion; the distinct not_found vs. already-done cases; the required invalid-input test per tool.

### Publishing as a completion criterion
**What.** "A stranger can install and use this without talking to me" is part of done, not a stretch.
**Why it exists.** A working local demo and an installable, documented, registry-listed package are different engineering-rigor levels; only the second is a credible, checkable claim.
**Where here.** PyPI + the official MCP Registry `server.json` submission.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Inspector as a validation tool | The habit of validating against a spec |
| 1 | Wrapping a flaky dependency safely (cache, typed errors) | Any "wrap API X as a tool" job task |
| 2 | Designing a search tool's interface (tool vs. resource) | API-design judgment beyond MCP |
| 3 | Idempotency + CRUD semantics under a tool interface | Universal backend skill in an agent context |
| 4 | Proving an interop claim, not asserting it | The habit that separates impressive-sounding from verifiably-true |
| 5 | Protocol/transport-level testing + real publishing | A PyPI package + passing CI beat a screenshot |
| 6 | Docs for a stranger | The real bottleneck in "share your server" is docs, not code |

## 4. Common Misconceptions & Mistakes

- **Local demo with your own client = compliant.** Only cross-client verification proves it.
- **Skipping error-path tests.** "Works on good input" says little about safety to depend on.
- **Tools/resources conflated.** A read-only listing (`task_board`) is a resource, not a no-arg tool.
- **"Published" = "on GitHub."** PyPI + registry is stronger.
- **Over-building one server into a mini-app.** Redundant with Project 10.

## 5. Understanding-check questions (with answer key)

**Q1 (Authorship).** What design decisions does authoring force that consuming never does? One concrete example from your servers.
**A1.** Whether a capability is a tool or a resource; the error taxonomy; idempotency semantics. Example: for the task-tracker you must decide that `complete_task` on an already-done task returns it unchanged (idempotent) while an unknown id is a typed not_found error — a consumer never makes that call.

**Q2 (Compliance).** Why is "works with my own client" insufficient? What could your own client never notice?
**A2.** Your client might always send an optional field, or tolerate a slightly-wrong schema, or use one transport. A spec-compliant third-party client might omit that field, strictly validate the schema, or use a different transport — exposing assumptions your client masked.

**Q3 (Idempotency/errors).** For the task-tracker, how should `complete_task` behave on a nonexistent id vs. an already-completed one? Why differently?
**A3.** Nonexistent id → a typed not_found error (the caller referenced something that doesn't exist — a real problem). Already-completed → return the task unchanged (idempotent success — the desired end state already holds, retries must be safe). Conflating them either hides real errors or turns safe retries into failures.

**Q4 (Publishing).** Difference in practice between "I built" and "I published" an MCP server? Which is on your resume?
**A4.** "Built" = code exists and runs for you. "Published" = a stranger runs `pip install`, follows your README with no undocumented steps, and it works — plus it's discoverable in the registry. The resume says "published," and the PyPI/registry link backs the word.

**Q5 (Cross-client).** A different client fails to connect to your knowledge-store server. First two things to check, given tool/resource typing?
**A5.** (1) Schema validity — did FastMCP generate a valid JSON Schema from your type hints (e.g., an unsupported/undocumented type on a param)? Run Inspector. (2) Tool-vs-resource mismatch — is `document_index` declared as a resource but the client expects a tool, or vice versa, or an unclamped `top_k` causing a validation error? Check the handshake/capability negotiation and the exact error the client reports.
