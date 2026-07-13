# PROFESSOR-NOTES.md — A2A Multi-Framework Agent Network

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Project 01 built (or at least its worker designs understood) | You're re-implementing 3 of its specialists in new frameworks | Your own Project 01 |
| Basic CrewAI (agents, tasks, crews) | You're building one specialist in CrewAI | CrewAI's own quickstart docs |
| Basic Agno (agents, tools) | You're building one specialist in Agno | Agno's own quickstart docs |
| MCP fundamentals (same as Project 04) | Every specialist here also exposes an internal MCP tool | Reuse what you learned building Project 04, or read it first if you haven't |
| What "service discovery" means in distributed systems | The Coordinator's whole design rests on this idea, generalized to agents | Any short primer on service discovery patterns (even outside AI — the concept is the same) |

## 2. Core Concepts Taught

### Agent-to-Agent (A2A) protocol
**What:** an open protocol (from Google) for agents built on *different* frameworks or by *different* teams to discover each other's capabilities and delegate tasks, without either side needing to know the other's internal implementation.
**Why it exists:** MCP solves "how does an agent talk to a tool"; A2A solves the complementary problem of "how does one agent talk to another agent" — especially across organizational or framework boundaries, where you can't just import someone else's Python class. As agentic systems multiply, no single team builds every specialist agent in the same framework, so a common inter-agent protocol matters.
**How it works:** each agent publishes an **agent card** — a small, structured self-description of its capabilities and how to reach it. A requesting agent (or coordinator) fetches the card, then sends a task request in the protocol's standard shape and receives a task response, entirely independent of what's inside the responding agent.
**Where it's used here:** every specialist agent's A2A server wrapper, and the Coordinator's discovery-then-dispatch logic — the core subject of this whole project.

### Framework-agnostic orchestration
**What:** designing a coordinator/orchestrator that can call agents built in different frameworks interchangeably, because it only depends on the standard protocol boundary (A2A), never on framework-specific internals.
**Why it exists:** real organizations don't standardize on one agent framework forever — teams pick different tools, acquire other teams' systems, or migrate frameworks over time. An orchestrator that can't survive a specialist being rebuilt in a different framework is brittle in exactly the way real production systems can't afford to be.
**Where it's used here:** the Phase 5 "swap one specialist's framework, zero coordinator changes" proof — the single most important validation in this project.

### MCP and A2A used together
**What:** using MCP for an agent's own tool access and A2A for that same agent's exposure to other agents, in the same system.
**Why it exists:** these protocols solve adjacent but distinct problems and are commonly discussed together as "the 2026 agent protocol stack" — most portfolio projects use at most one; using both correctly, and being able to clearly explain which does what, is the specific signal this project is built to produce.
**Where it's used here:** every specialist agent in this project's architecture diagram — MCP arrows going "down" to tools, A2A arrows going "sideways/up" to the coordinator.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Minimal A2A round-trip mechanics | De-risks the rest of the build by isolating protocol plumbing first |
| 1 (LangGraph agent) | Wrapping an existing LangGraph agent for external A2A exposure | Reusable pattern for exposing any of your other portfolio projects' agents externally |
| 2 (CrewAI agent) | A second framework's idioms, under real time pressure to keep it simple | Breadth signal — most candidates only ever show one framework |
| 3 (Agno agent) | A third framework, plus MCP tool-wrapping inside it | Completes the "framework fluency" story |
| 4 (Coordinator) | Discovery-based orchestration design | The actual architecture pattern behind "an agent marketplace/ecosystem," an increasingly real enterprise need |
| 5 (Interop proof) | Proving an abstraction boundary holds by actually exercising it | A rare, concrete engineering-rigor signal — most projects claim modularity without ever testing it |
| 6 (Polish) | Telling a protocol-literacy story clearly to a non-specialist reader | This project's pitch is subtle; the README has to do real explanatory work |

## 4. Common Misconceptions & Mistakes

- **Confusing MCP and A2A.** MCP = agent-to-tool. A2A = agent-to-agent. Mixing these up out loud in an interview undercuts the exact credibility this project is meant to build — know the distinction cold.
- **Building 3 frameworks that never really need to be separate services.** If everything runs in one process and could've been 3 plain functions, you've demonstrated framework syntax, not interoperability.
- **Skipping the swap-and-verify step.** Claiming "this design would support swapping frameworks" without ever doing it is a much weaker claim than actually doing it once and documenting the diff.
- **Hardcoding an endpoint "just to get the demo working" and forgetting to remove it.** This silently breaks the discovery-based design the whole project is supposed to demonstrate.
- **Ignoring or hiding the latency cost of going over-the-wire.** Every real architecture decision has a tradeoff; hiding the number looks less credible than reporting and explaining it.

## Understanding-check questions

**After §2 (A2A protocol):** What problem does A2A solve that MCP does not? Could you build this project's Coordinator using only MCP? Why or why not?

**After §2 (Framework-agnostic orchestration):** Why does the Coordinator's ability to work with a swapped-out specialist matter more than the specific frameworks you chose? What would you tell an interviewer who asks "why these 3 frameworks specifically"?

**After §2 (MCP + A2A together):** Draw (in words) the two different arrows in this project's architecture — one for MCP, one for A2A. What's flowing over each, and between which parties?

**After Phase 4 (Coordinator):** What's one design decision in your Coordinator that would leak framework-specific knowledge if you weren't careful, and how did you avoid it?

**After Phase 5 (Interop proof):** You measured A2A overhead vs. an in-process call and it's 5x slower. Is that a problem with this design, or an expected and acceptable tradeoff? What would change your answer?
