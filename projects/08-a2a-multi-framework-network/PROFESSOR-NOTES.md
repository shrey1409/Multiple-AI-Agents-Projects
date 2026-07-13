# PROFESSOR-NOTES.md — A2A Multi-Framework Agent Network

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Project 01 built (or its worker designs understood) | You re-implement 3 of its specialists | Your own Project 01 |
| Basic CrewAI (agents/tasks/crews) | You build one specialist in CrewAI | CrewAI quickstart docs |
| Basic Agno (agents/tools) | You build one specialist in Agno | Agno quickstart docs |
| MCP fundamentals | Every specialist exposes an internal MCP tool | Reuse Project 04 |
| Service discovery (distributed systems) | The coordinator's design rests on it | Any service-discovery primer; the concept is the same outside AI |
| JSON-RPC 2.0 | A2A interactions are JSON-RPC | JSON-RPC 2.0 spec summary |

## 2. Core Concepts Taught

### Agent-to-Agent (A2A) protocol
**What.** An open protocol (originally Google, now under the Linux Foundation) for agents on *different* frameworks/teams to discover each other and delegate tasks without knowing internals.
**Why it exists.** MCP solves agent→tool; A2A solves agent→agent, especially across framework/organizational boundaries where you can't just import someone's class.
**How it works (mechanism).** Each agent publishes an **agent card** at `/.well-known/agent-card.json` (RFC 8615 well-known URI) describing its `skills`, `capabilities`, and `url`. A client fetches the card, then sends a `message/send` JSON-RPC request containing a `Message` (made of `Part`s); the server returns a `Task` that progresses through states `submitted → working → (input-required) → completed|failed`. The client can `tasks/get` to poll. All of this is independent of what's inside the responding agent.
**Where here.** Every specialist's A2A server wrapper + the coordinator's discovery-then-dispatch.

### Framework-agnostic orchestration
**What.** A coordinator that calls agents built in different frameworks interchangeably because it depends only on the A2A boundary.
**Why it exists.** Real orgs don't standardize on one framework forever — teams differ, migrate, get acquired. An orchestrator that survives a specialist being rebuilt in another framework is what production actually needs.
**Where here.** The Phase-5 swap-with-zero-coordinator-changes proof — the most important validation.

### MCP and A2A together
**What.** MCP for an agent's own tool access; A2A for that agent's exposure to other agents — in one system.
**Why it exists.** They solve adjacent-but-distinct problems and are the "2026 agent protocol stack." Using both correctly and explaining which does what is the specific signal this project produces.
**Where here.** Every specialist: MCP arrows go "down" to tools, A2A arrows go "sideways" to the coordinator.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Minimal A2A round-trip | De-risks the build by isolating protocol plumbing |
| 1 | Wrapping a LangGraph agent for external A2A | Reusable for exposing any portfolio agent externally |
| 2 | A second framework under time pressure | Breadth signal — most candidates show one framework |
| 3 | A third framework + MCP tool inside it | Completes the fluency story |
| 4 | Discovery-based orchestration | The pattern behind an "agent marketplace/ecosystem" |
| 5 | Proving an abstraction boundary by exercising it | A rare engineering-rigor signal |
| 6 | Explaining protocol literacy to a non-specialist | This project's pitch is subtle; the README must do real work |

## 4. Common Misconceptions & Mistakes

- **Confusing MCP and A2A.** MCP = agent→tool; A2A = agent→agent. Know it cold.
- **3 frameworks that never needed to be separate services.** Then you showed syntax, not interop.
- **Skipping the swap-and-verify.** "Would support swapping" is far weaker than doing it once and diffing.
- **Hardcoding an endpoint and forgetting it.** Silently breaks discovery.
- **Hiding the latency cost.** Report and explain the tradeoff.

## 5. Understanding-check questions (with answer key)

**Q1 (A2A).** What does A2A solve that MCP doesn't? Could you build the coordinator with only MCP?
**A1.** A2A handles agent-to-agent task delegation across framework/org boundaries (discovery via agent cards, task lifecycle). MCP handles agent-to-tool access. You could technically expose an agent *as* an MCP tool, but you'd lose A2A's task lifecycle, agent-card capability discovery, and the peer semantics — MCP models "a tool the client drives," not "a peer agent you delegate a task to and poll."

**Q2 (Framework-agnostic).** Why does surviving a swapped specialist matter more than the specific frameworks? What do you tell an interviewer asking "why these 3"?
**A2.** The value is the *boundary*, not the frameworks — production needs orchestration that outlives any one framework choice. To the interviewer: "The three are just to prove heterogeneity; the real result is that the coordinator didn't change when I swapped one, which is the property real systems need."

**Q3 (MCP + A2A).** Describe the two arrows in this architecture — what flows over each, between whom?
**A3.** MCP arrow: from a specialist agent *down* to its tool (e.g., the market-data agent → its yfinance MCP tool), carrying tool calls/results. A2A arrow: from the coordinator *sideways* to a specialist agent, carrying a `Message`/`Task` (a delegated request and its result).

**Q4 (Coordinator).** Name one place framework-specific knowledge could leak and how you avoided it.
**A4.** Response parsing: if the coordinator special-cased "if it's the CrewAI agent, read field X," it would leak. Avoided by having every specialist return the same A2A `Task`/`Message` shape, so the coordinator parses one uniform structure regardless of backing framework.

**Q5 (Interop proof).** A2A overhead is 5× an in-process call. Problem or acceptable? What changes your answer?
**A5.** Usually acceptable — you bought framework independence and network deployability with that latency, and the calls are I/O-bound anyway (LLM + external APIs dominate). It becomes a problem if the workload is latency-critical and high-QPS where the per-hop overhead dominates total time; then you'd co-locate or batch. The answer depends on the absolute latency budget, not the ratio alone.
