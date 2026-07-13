# PROFESSOR-NOTES.md — MCP Server Trilogy

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| MCP fundamentals (tools/resources/prompts, transports) | This entire project is authoring MCP servers | If you built Project 04 first, you already have this — otherwise read the official MCP spec before starting |
| Basic package publishing (what a `pyproject.toml` is, what `pip install` from PyPI requires) | Phase 5/6 require actually publishing a package | PyPI's own "Packaging Python Projects" tutorial |
| Writing tests that spin up a real subprocess (not just unit-testing internal functions) | Your test suite must exercise the real protocol transport | Python's `subprocess` module docs, or your test framework's fixture patterns for external processes |

## 2. Core Concepts Taught

### Tool-provider authorship as a distinct skill
**What:** designing and building the *server* side of a tool-access protocol — deciding what capabilities to expose, how to type their inputs/outputs, and how to handle failure — as opposed to consuming someone else's already-built server.
**Why it exists:** most "I built an AI agent" projects only exercise the consumer side (call this existing tool); authoring a well-designed server requires API-design judgment (what's a tool vs. a resource, what should be idempotent, what error shape is useful to a caller) that consuming tools never forces you to practice.
**Where it's used here:** every one of the 3 servers — this project exists specifically to isolate and practice this skill.

### Protocol compliance vs. "it works for me"
**What:** the difference between a server that happens to work when called by the one client you wrote, and a server that's verifiably correct against the protocol's actual specification — checkable by an independent tool (MCP Inspector) or a different client entirely.
**Why it exists:** a server tightly coupled to assumptions baked into your own test client is not actually reusable — the entire value of building on a standard protocol is that *anyone's* compliant client can use your server, and that claim is only as strong as your verification of it.
**Where it's used here:** the Phase 4 cross-client verification step — the single most important validation in the whole project, more convincing than any amount of testing against your own client.

### Idempotency and error semantics as API design, not an afterthought
**What:** deciding, deliberately, what should happen when a tool is called twice with the same effective input, or with invalid input — and making that behavior predictable and typed rather than accidental.
**Why it exists:** a tool provider that crashes on bad input, or produces different results on retries of "the same" action, is unpleasant and unsafe to build agents against — this is API design 101, applied to the MCP context.
**Where it's used here:** `complete_task`'s idempotent-completion behavior, and the required invalid-input test case for every tool in §6 of PLAN.md.

### Publishing as a completion criterion
**What:** treating "can a stranger install and use this without talking to me" as part of the definition of done, not a stretch goal.
**Why it exists:** a working demo on your own machine and a genuinely installable, documented package are different levels of engineering rigor, and only the second is a credible, checkable portfolio claim.
**Where it's used here:** the PyPI packaging and community-directory-submission requirements in Phase 5/6.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | MCP Inspector as a validation tool, not just a demo toy | The habit of validating against a spec, not just against your own expectations |
| 1 (API Wrapper) | Wrapping a flaky external dependency safely (caching, error typing) | Directly transferable to any "wrap API X as a tool" task in a real job |
| 2 (Knowledge Store) | Designing a search tool's interface (what's a tool vs. a resource) | API-design judgment that generalizes beyond MCP specifically |
| 3 (Task Tracker) | Idempotency and CRUD semantics under a tool-call interface | A universal backend-engineering skill, reinforced in an agent context |
| 4 (Cross-client verification) | Proving an interoperability claim instead of asserting it | The single habit that most separates "sounds impressive" projects from verifiably true ones |
| 5 (Tests+Packaging) | Testing at the protocol/transport level, real package publishing | Concrete artifacts (a PyPI package, a passing CI suite) beat a screenshot every time |
| 6 (Publish) | Writing documentation for a stranger, not for yourself | The actual bottleneck in most "share your MCP server" attempts is documentation, not code |

## 4. Common Misconceptions & Mistakes

- **Thinking a working demo with your own client is the same as a compliant server.** It isn't — only cross-client verification proves that.
- **Skipping error-path tests.** "It works when I give it good input" tells you almost nothing about whether the server is safe to depend on.
- **Conflating tools and resources.** A read-only listing (like `task_board`) is a better fit for MCP's `resource` concept than forcing it into a `tool` that happens to take no meaningful arguments — this distinction is explicitly tested in the understanding-check questions.
- **Treating "published" as equivalent to "code is on GitHub."** A real package on PyPI, installable by a stranger with one command, is a meaningfully stronger and different claim.
- **Over-building one server into a mini-app.** The Knowledge-Store server needs a good `search_documents`/`get_document` pair, not a web UI or a full production RAG pipeline — that would be redundant with Project 10 anyway.

## Understanding-check questions

**After §2 (Tool-provider authorship):** What design decisions does authoring a server force you to make that consuming an existing tool never does? Give one concrete example from one of your 3 servers.

**After §2 (Protocol compliance):** Why is "it works when my own test client calls it" insufficient evidence that a server is protocol-compliant? What specifically could be wrong that your own client would never notice?

**After §2 (Idempotency/error semantics):** For your Task-Tracker server, what should happen if `complete_task` is called on a task ID that doesn't exist at all (vs. one that's already completed)? Why should these two cases behave differently?

**After §2 (Publishing):** What's the difference, in practice, between "I built an MCP server" and "I published an MCP server"? Which one is on your resume, and can you back up that specific word?

**After Phase 4 (Cross-client verification):** You try connecting a different MCP client to your Knowledge-Store server and it fails. What are the first two things you'd check, given what you know about tool/resource typing?
