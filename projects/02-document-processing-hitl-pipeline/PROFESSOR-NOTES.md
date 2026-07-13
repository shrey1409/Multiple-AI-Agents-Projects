# PROFESSOR-NOTES.md — Document-Processing Pipeline with Human-in-the-Loop

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| LangGraph `StateGraph` + checkpointers | This is one graph with a suspend/resume point | LangGraph docs → "Persistence" and "Human-in-the-loop" concept pages |
| Pydantic structured extraction | Extraction's whole job is typed field extraction | LangChain "How to do extraction" how-to |
| Postgres transactions + unique constraints | Your audit log and idempotency guard live here | PostgreSQL docs → "Constraints" (UNIQUE) and "Transactions"; you need only these |
| Webhook signature verification | The Slack callback is a security surface | Slack "Verifying requests from Slack" |

## 2. Core Concepts Taught

### Durable execution / interrupts (`interrupt()`)
**What.** Pause a running graph at a node, persist its exact state, resume later — possibly from a different process, hours/days on.
**Why it exists.** Human approval can take minutes to days. You cannot hold a thread open that long, and "poll a status column in a loop" is racy and wasteful. Durable execution makes "waiting for a human" a first-class persisted state.
**How it works (mechanism).** The interrupt node raises a signal the checkpointer catches; it serializes state to storage (Postgres) keyed by `thread_id` and returns control to the caller. A later `graph.invoke(Command(resume=decision), config={"configurable": {"thread_id": document_id}})` rehydrates that state and continues. Because state lives in Postgres, a full process restart in between is fine.
**Where here.** The Human Approval node — the reason Postgres-backed (not in-memory) checkpointing is mandatory.

### Human-in-the-loop (HITL) design
**What.** Defer consequential decisions to a human based on **confidence and risk**, not confidence alone.
**Why it exists.** Autonomous action on irreversible/high-stakes outcomes is where agents cause real damage; HITL is the industry mitigation, and building one correctly (not "email someone") is a specific hireable skill.
**How it works.** Validation scores confidence and risk → routes to auto-execute or suspend-and-notify → the human decision is captured as data (who/when/what) feeding the same state the auto path would produce.
**Where here.** The `requires_human` branch, combining confidence and business-impact rules.

### Idempotency (and why exactly-once is a myth)
**What.** Designing an operation so doing it N times equals doing it once.
**Why it exists.** Distributed systems retry on failure by default (network blips, crashes). **Exactly-once delivery is impossible** across a network — you cannot atomically "do the side effect" and "record that you did it" in the outside world. The achievable pattern is **at-least-once delivery + idempotent handlers**: retries may re-deliver, but the idempotency key makes re-execution a no-op.
**How it works.** Derive a stable key from inputs (`sha256(document_id + action_type)`), and make check-and-execute atomic via a Postgres UNIQUE constraint (INSERT … ON CONFLICT), **not** an application-level read-then-write (which races). The transactional-outbox pattern generalizes this when the side effect is "send a message".
**Where here.** The Action Executor + the replay/concurrency tests.

### Audit logging as a design constraint
**What.** An append-only record of every meaningful transition, enough to reconstruct "what happened and why".
**Why it exists.** Regulated domains (finance, healthcare, insurance — exactly what this simulates) must answer "why did the system do X". This is a hard requirement, not a nice-to-have.
**Where here.** The Audit Logger writing to a separate `audit_log` table on every transition, independent of graph state (survives schema changes). Note it logs *findings/metadata*, never more raw PII than necessary.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Schema design for auditability | Enterprise agent work is 30% "can this survive an audit" |
| 1 | Multi-class extraction + real confidence (self-consistency) | Directly transferable to document-AI roles; most candidates fake confidence |
| 2 | Deterministic logic vs. probabilistic LLM output | The #1 junior mistake is not knowing when *not* to use an LLM |
| 3 | Durable suspend/resume | The pattern behind every production "approve this agent action" |
| 4 | Idempotency keys, safe retries, at-least-once semantics | Universal distributed-systems skill, shows up in backend interviews too |
| 5 | Recall vs. precision tradeoffs for a safety-critical route | Report two numbers, not one average — a maturity signal |
| 6 | Timeouts for human-dependent steps | Real HITL dies on "what if the human never responds" |
| 7 | Communicating a safety-oriented design | "I built the approval-gate pattern enterprises ship" differentiates you |

## 4. Common Misconceptions & Mistakes

- **"HITL = add a Slack message."** The hard 90% is suspend/resume + the idempotency guarantee around the action; the notification is the easy 10%.
- **LLM validation instead of rules.** "The model thought so" is not an auditable answer.
- **Confidence as risk.** Different axes; conflating them rubber-stamps high-confidence high-stakes actions.
- **Skipping the restart-mid-approval test.** Separates real durable execution from a demo that breaks on restart.
- **One global timeout.** Different doc types have different urgency.
- **Read-then-write idempotency.** Not atomic under concurrency — two requests both pass the check.

## 5. Understanding-check questions (with answer key)

**Q1 (Durable execution).** What breaks with an in-memory checkpointer? Walk the restart scenario.
**A1.** In-memory state lives in the process. Suspend at approval → process restarts (deploy, crash) → state is gone → the resume webhook has no thread to resume → the document is lost or must restart from scratch, re-doing side effects. Postgres-backed state survives the restart.

**Q2 (HITL).** Give a low-confidence-low-risk doc and a high-confidence-high-risk doc. Why treat them differently?
**A2.** A $12 invoice extracted at 60% (low conf, low risk) can auto-process — a wrong $12 is cheap. A $2M invoice at 95% (high conf, high risk) should still be reviewed — the cost of a rare error is huge. Routing on confidence alone rubber-stamps the second.

**Q3 (Idempotency).** Why isn't "check if key exists, then insert" idempotent under concurrency, and what fixes it?
**A3.** Two requests can both read "absent" before either inserts, then both execute. A Postgres UNIQUE constraint with INSERT … ON CONFLICT makes the check-and-claim a single atomic operation; the loser gets a conflict and skips the side effect.

**Q4 (Validation).** Why rules, not an LLM? When would an LLM be appropriate, and how keep it auditable?
**A4.** Rules are exact and explainable (an auditor sees "amount_over_10k fired"). An LLM helps for genuinely fuzzy judgments (e.g., "does this contract clause look non-standard") — keep it auditable by having it output a *structured reason + the clause span*, logged, and still gate the final action on a deterministic rule.

**Q5 (Escalation).** Design escalation for two doc types with different urgency. What data sets the timeout?
**A5.** Config per doc_type: invoice 48h → escalate to a second reviewer; time-sensitive claim 4h. Data needed: doc_type, submission timestamp, business SLA per type, and reviewer availability/on-call schedule.
