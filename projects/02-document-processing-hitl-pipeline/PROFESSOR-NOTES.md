# PROFESSOR-NOTES.md — Document-Processing Pipeline with Human-in-the-Loop

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| LangGraph `StateGraph` basics | This project is one graph with a suspend/resume point | LangGraph Quickstart (verify against your installed version — see RESOURCES.md) |
| Structured extraction with Pydantic | Extraction agent's whole job is typed field extraction | LangChain's structured-output / extraction how-to guides |
| Basic Postgres (tables, transactions, unique constraints) | Your audit log and idempotency guard live here | Any "Postgres for application developers" primer — you need transactions and unique constraints, not much else |
| Webhooks (what they are, how to verify a signature) | The Slack approval callback is a webhook you must secure | Slack's own "Verifying requests from Slack" doc |

## 2. Core Concepts Taught

### Durable execution / interrupts (LangGraph `interrupt()`)
**What:** a mechanism to pause a running graph at a specific node, persist its exact state, and resume it later — potentially from a different process, hours or days after the pause.
**Why it exists:** a human approval step can take minutes to days. You cannot hold a thread/process open that whole time, and a naive "poll a status column in a loop" approach is racy and wastes resources. Durable execution treats "waiting for a human" as a first-class state, not a workaround.
**How it works:** the interrupt node raises a special signal that LangGraph's checkpointer catches; it serializes the current state to storage and returns control to the caller. A later, separate call (`graph.invoke(Command(resume=human_decision), config={"thread_id": ...})`) rehydrates that exact state and continues.
**Where it's used here:** the Human Approval node — the single most important piece of this project's design, and the reason Postgres-backed (not in-memory) checkpointing is mandatory, not a nice-to-have.

### Human-in-the-loop (HITL) design
**What:** a system design pattern where an automated pipeline defers consequential decisions to a human reviewer instead of acting autonomously, based on confidence and risk thresholds.
**Why it exists:** fully autonomous action on irreversible or high-stakes outcomes (money movement, legal commitments) is where AI agents cause the most real-world damage; HITL is the industry-standard mitigation, and building one correctly (not just "email someone") is a hireable, specific skill.
**How it works:** a validation step scores confidence/risk and routes to either auto-execute or a suspend-and-notify path; the human's decision is captured as data (who, when, what) and feeds back into the same state the automated path would have produced.
**Where it's used here:** the `requires_human` branch and the whole HITL node — note it is deliberately combined with confidence *and* business-impact rules (§7 risk), not confidence alone.

### Idempotency
**What:** designing an operation so that performing it multiple times has the same effect as performing it once.
**Why it exists:** distributed systems retry on failure by default (network blips, process crashes); without idempotency, retries after a partial failure cause duplicate side effects — double payments, duplicate tickets.
**How it works:** derive a stable key from the operation's inputs (content hash + action type here), check for that key's existence before executing, and make the check-and-execute atomic (a unique constraint in Postgres, not an application-level "check then act" race).
**Where it's used here:** the Action Executor and the idempotency replay test in §6/§9 of PLAN.md.

### Audit logging as a design constraint, not an afterthought
**What:** an append-only record of every meaningful state transition, sufficient to reconstruct "what happened and why" after the fact.
**Why it exists:** enterprises adopting agentic automation for consequential actions need to answer "why did the system do X" for compliance and debugging — this is a hard requirement in regulated domains (finance, healthcare, insurance), which is exactly the domain this project simulates.
**Where it's used here:** the Audit Logger writing to a separate `audit_log` table on every transition, independent of the graph's own state (so it survives even if you later change the state schema).

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Schema design for auditability | Enterprise agent work is 30% "can this survive an audit," not just "does it work" |
| 1 (Classify+Extract) | Multi-class structured extraction with discriminated unions | Directly transferable to any document-AI job posting |
| 2 (Validation) | Separating deterministic business logic from probabilistic LLM output | The single most common mistake junior agent engineers make is not knowing when *not* to use an LLM |
| 3 (HITL Interrupt) | Durable execution / suspend-resume design | This is the pattern behind every production "approve this agent action" feature at real companies |
| 4 (Action+Idempotency) | Idempotency keys, safe retries | A universal distributed-systems skill, not agent-specific — shows up in every backend interview too |
| 5 (Eval) | Measuring recall vs. precision tradeoffs for a safety-critical routing decision | Teaches you to report two numbers instead of one misleading average — a maturity signal to reviewers |
| 6 (Deploy+Escalation) | Designing timeouts for human-dependent steps | Real HITL systems die when nobody thinks about "what if the human never responds" |
| 7 (Polish) | Communicating a safety-oriented system's design tradeoffs | This project's story ("I built the approval-gate pattern enterprises actually ship") is a strong differentiator in interviews |

## 4. Common Misconceptions & Mistakes

- **"HITL just means add a Slack message."** The hard part is the suspend/resume semantics and the idempotency guarantee around the eventual action — the notification is the easy 10%.
- **Using an LLM to "validate" instead of fixed rules.** If a human asks "why was this flagged," "the model thought so" is not an acceptable answer in a regulated workflow. Deterministic rules are a feature, not a limitation.
- **Treating confidence score as risk score.** They are different axes; conflating them creates a system that rubber-stamps high-confidence-but-high-stakes actions.
- **Skipping the "process restarted mid-approval" test.** This is the scenario that separates a real durable-execution implementation from a demo that only works if the server never restarts.
- **One global timeout for all escalations.** Different document types have different urgency; hardcoding one timeout either annoys reviewers (too short) or lets stale approvals rot (too long).

## Understanding-check questions

**After §2 (Durable execution):** What specifically would break in this project if you used an in-memory checkpointer instead of a Postgres-backed one? Walk through the process-restart scenario.

**After §2 (HITL design):** Give an example of a document that would have low extraction confidence but low risk, and one with high confidence but high risk. Why does your validation logic need to treat these differently?

**After §2 (Idempotency):** Why is "check if the key exists, then insert" not truly idempotent under concurrent requests, and what Postgres feature fixes this?

**After Phase 2 (Validation):** Why does the plan insist the rules engine is plain code, not an LLM call? Can you think of a business rule where an LLM actually would be more appropriate — and how would you keep that decision auditable anyway?

**After Phase 6 (Escalation):** Design the escalation policy for two document types with very different urgency. What data would you need to set a sensible timeout for each?
