# PROFESSOR-NOTES.md — Agent Guardrail Gateway

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| A target agent (Project 01/02) | You need something to protect, black-box | Your own build |
| NER (named entity recognition) | PII redaction relies on it | spaCy "Linguistic Features → Named Entities"; you use a model, don't train one |
| Prompt injection (deeper than Project 03) | You build the defense, not just simulate | OWASP LLM Top 10 → LLM01, read in depth |
| Rate limiting (token bucket / sliding window) | The limiter needs it | Any rate-limiting-algorithms primer |
| Race conditions + atomic ops | The budget bug in §7 is a real concurrency class | A primer on why read-then-write isn't atomic; what `INCR` solves |

## 2. Core Concepts Taught

### Middleware/gateway pattern for AI safety
**What.** Insert a protective layer *between* callers and an agent, rather than baking safety into the agent.
**Why it exists.** Safety inside one agent only protects that agent and mixes safety with business logic. A separate gateway protects *any* agent behind it, updates independently, and is a far stronger "reusable safety product" claim.
**Where here.** The whole gateway — agent-agnostic, verified against 01/02 as an external black box.

### PII redaction
**What.** Detect and mask personal info before it's sent onward or logged.
**Why it exists.** Agents risk leaking PII into logs, a provider's data, or downstream systems; gateway-layer redaction is defense-in-depth.
**How it works (mechanism).** Two complementary detectors: **regex** for structured PII (emails, SSNs, card numbers — with a Luhn checksum to cut false positives on random digit runs) and **NER** for unstructured PII (person/org/place names regex can't enumerate). Log *findings* (types), never values.
**Where here.** Inbound + outbound PII Redaction.

### Prompt-injection defense
**What.** Detect attempts to override the system's instructions via input text, and block/flag before they reach the target.
**Why it exists.** LLMs don't natively separate "my system-prompt instructions" from "instructions embedded in user content" (OWASP ranks this #1). A gateway adds a layer the model doesn't provide. It's inherently imperfect — injection is an open adversarial space — so you scope the claim to a *named pattern set* and report both block and false-positive rates.
**Where here.** The heuristic-first, LLM-secondary detector.

### Structural safety for spend and actions
**What.** The least-privilege / can't-happen-structurally principle (Project 05's read-only DB, Project 02's idempotency) applied to budgets and to which actions need approval.
**Why it exists.** An agent that can spend unboundedly or take any action is a real financial/operational risk; hard limits at a layer the agent can't talk around (a gateway, not a prompt) are the reliable mitigation.
**How it works (mechanism).** The budget check is an **atomic** Redis `INCR`-and-compare — two concurrent requests can't both pass because the increment and the check are one operation, unlike read-then-write. The allow-list is deterministic code, not an LLM judgment.
**Where here.** The Spend/Rate Limiter and Allow-List Checker.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | A transparent proxy in front of an arbitrary API | A general backend pattern |
| 1 | NER application, recall/precision tradeoffs | Any "handle user data safely" requirement |
| 2 | Layered defense: cheap heuristic + calibrated LLM check | Cost-aware defense design |
| 3 | Atomic counters, race-aware design | Universal distributed-systems skill + LLM cost control |
| 4 | HITL in a security-boundary context | Generalizing a pattern across domains |
| 5 | Designing adversarial + FP test sets | Evaluating a *defense* (different from evaluating task success) |
| 6 | Communicating limitations honestly | Overclaiming "100% safe" is a red flag to reviewers |

## 4. Common Misconceptions & Mistakes

- **Any single layer is sufficient.** PII, injection, and rate limits address different risks; defense-in-depth is the lesson.
- **Coupling to one target's internals.** Import its API, not its objects.
- **Logging PII "for debugging."** Log findings/metadata only.
- **Block rate without FP rate.** A block-everything detector "wins" on block rate and is useless.
- **Read-then-write budget checks.** A classic race; use atomic ops.

## 5. Understanding-check questions (with answer key)

**Q1 (Gateway).** Why is a separate gateway a stronger claim than a safety check inside one agent? What proves it's genuinely reusable?
**A1.** A gateway protects any agent behind it and can be updated independently of business logic. Proof of reusability: point it at a *second*, different agent (01 and 02) via config only, with no gateway code changes — the same proof pattern as Projects 08/09/12.

**Q2 (PII).** Why log findings, not redacted values? What goes wrong logging the PII "just in case"?
**A2.** The audit log would become a second store of the sensitive data you're trying to protect — a new leak vector with its own access/retention risk. Logging only the *types* found preserves auditability ("an email was redacted here") without duplicating the PII.

**Q3 (Injection).** Why is FP rate as important as block rate? A legit request a naive detector wrongly blocks?
**A3.** A detector optimized only for block rate can block everything and score 100% while being unusable. Example FP: a user legitimately says "ignore the previous recommendation and show me a cheaper option" — surface-similar to an injection, but benign; blocking it frustrates real users.

**Q4 (Structural safety).** Why is an atomic-counter budget check "structural" like Project 05's read-only role, while a prompt "don't spend >$X" is not?
**A4.** The atomic counter makes over-budget requests *impossible* to pass regardless of what the agent "wants" — enforcement lives in infrastructure the agent can't influence. A prompt instruction is advisory; the agent can be manipulated or simply ignore it, so it's not a boundary.

**Q5 (Rate limiting).** Design a test that catches a race in budget enforcement. What pattern, and what's pass vs. fail?
**A5.** Fire N concurrent requests whose combined cost just exceeds the budget. Pass: exactly the requests that fit under budget succeed and the rest are blocked (total spend ≤ budget). Fail: two requests both read "under budget" before either increments and both proceed, so total spend exceeds the limit — the read-then-write race.
