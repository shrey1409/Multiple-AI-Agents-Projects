# PROFESSOR-NOTES.md — Agent Guardrail Gateway

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| A target agent already built (Project 01 or 02) | You need something to protect, treated as a black box | Your own Project 01/02 build |
| Basic NER (named entity recognition) concepts | Your PII redaction layer relies on this | Any short primer on NER — you don't need to train a model, just use one |
| Prompt injection, conceptually (same prerequisite as Project 03) | This project's core defensive feature | OWASP's LLM Top 10 entry on prompt injection — read it more deeply here than you did for Project 03, since you're building the defense, not just simulating the attack |
| Rate limiting / token-bucket algorithms | The spend/rate limiter needs this | Any short primer on rate-limiting algorithms (token bucket, sliding window) |
| Race conditions and atomic operations | The budget enforcement risk in §7 is a real concurrency bug class | A short primer on why "read then write" isn't atomic, and what `INCR`-style atomic operations solve |

## 2. Core Concepts Taught

### Middleware/gateway pattern for AI safety
**What:** inserting a protective layer *between* users/callers and an agent, rather than building safety checks into the agent itself.
**Why it exists:** safety logic embedded inside one specific agent only protects that agent, and mixes safety concerns with business logic; a separate gateway can protect *any* agent behind it, can be updated independently, and is a much more credible "I built a reusable safety product" claim.
**Where it's used here:** the entire gateway architecture — deliberately kept agent-agnostic, verified by working against Project 01/02 as a genuinely external, black-box target.

### PII redaction
**What:** detecting and masking personally identifiable information (names, emails, phone numbers, government ID numbers, etc.) in text before it's sent onward or logged.
**Why it exists:** agents that process user text risk leaking PII into logs, into a model provider's training data (depending on provider policy), or into unintended downstream systems — redaction at a gateway layer is a standard defense-in-depth practice.
**Where it's used here:** the PII Redaction components, both inbound and outbound — note the deliberate choice to log *findings* (what type of PII, where) rather than the PII itself, so the safety log doesn't become a new leak vector.

### Prompt-injection defense
**What:** detecting attempts to manipulate an LLM-based system via its input text (instructions embedded in user content designed to override the system's actual instructions), and blocking or flagging them before they reach the target agent.
**Why it exists:** prompt injection is one of the most-cited LLM-specific security risks (OWASP's LLM Top 10 lists it first) because LLMs don't natively distinguish "instructions from my system prompt" from "instructions embedded in user-supplied content" — a gateway-level defense adds a layer the model itself doesn't provide.
**Where it's used here:** the Prompt-Injection Detector, and specifically the honest reporting of both block rate *and* false-positive rate in the eval strategy — a defense that blocks everything (including legitimate requests) isn't actually useful.

### Structural safety boundaries, applied to spend and actions
**What:** the same "least privilege / can't happen structurally" principle from Project 05's read-only DB role and Project 02's idempotency guarantees, applied here to dollar/token budgets and to which actions an agent may take without human approval.
**Why it exists:** an agent that can spend unboundedly or take any action it decides to is a real operational and financial risk in production; enforcing hard limits at a layer the agent cannot talk its way around (a gateway, not a prompt instruction) is the reliable mitigation.
**Where it's used here:** the Spend/Rate Limiter and the Action Allow-List Checker — both implemented as deterministic code checks, explicitly not LLM judgment calls, for the same reasons articulated in Project 02's PROFESSOR-NOTES.md.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Building a transparent proxy in front of an arbitrary API | A generally useful backend-engineering pattern, not just an AI-specific one |
| 1 (PII Redaction) | Practical NER application, recall/precision tradeoffs | Directly relevant to any "handle user data safely" requirement in a real job |
| 2 (Injection Detection) | Defensive pattern-matching plus calibrated use of a secondary LLM check | Teaches cost-aware layered defense design, not just "throw an LLM at it" |
| 3 (Spend/Rate Limiting) | Atomic counters, race-condition-aware design | A universal distributed-systems skill that also happens to matter enormously for LLM cost control |
| 4 (Allow-List+Escalation) | Reusing the HITL pattern in a security-boundary context | Shows you can generalize a pattern (HITL) across different problem domains, not just apply it once |
| 5 (Red-team Eval) | Designing your own adversarial + false-positive test sets | The specific skill of evaluating a *defense*, which is a different exercise from evaluating an agent's task success |
| 6 (Polish) | Communicating security limitations honestly | A credibility signal — overclaiming "100% safe" is a red flag to any technical reviewer |

## 4. Common Misconceptions & Mistakes

- **Believing any single defense layer is sufficient.** PII redaction, injection detection, and rate limits each address a different risk; none of them make an agent "safe" on their own — defense in depth is the actual lesson.
- **Coupling the gateway to one target's internals.** If your gateway code imports Project 01's internal Python objects instead of calling its public API, you've built a feature of Project 01, not a reusable gateway.
- **Logging the PII itself "for debugging."** Defeats the purpose — log findings/metadata, never the actual sensitive values.
- **Reporting injection block rate without also reporting false positives.** A detector tuned to block everything trivially "wins" on block rate while being useless in practice; both numbers are required to make the claim meaningful.
- **Read-then-write budget checks.** A classic race condition — two concurrent requests can both pass a budget check before either one's spend is recorded, silently blowing through the limit; use atomic increment-and-check operations instead.

## Understanding-check questions

**After §2 (Middleware/gateway pattern):** Why is a separate gateway a stronger portfolio claim than a safety check built into one agent's own code? What would you have to prove to convince a skeptical reviewer the gateway is genuinely reusable?

**After §2 (PII redaction):** Why does this project log PII *findings* rather than the redacted values themselves in the audit trail? What would go wrong if you logged the actual PII "just in case you need to review it later"?

**After §2 (Prompt-injection defense):** Why is false-positive rate just as important to report as block rate? Describe a legitimate user request that a naive injection detector might wrongly block.

**After §2 (Structural safety for spend/actions):** Why is a token-bucket/atomic-counter check on the budget "structural" in the same sense as Project 05's read-only DB role, while a prompt instruction telling the agent "don't spend more than $X" is not?

**After Phase 3 (Spend/Rate Limiting):** Design a test that would catch a race condition in your budget enforcement. What request pattern would you send, and what would a passing vs. failing result look like?
