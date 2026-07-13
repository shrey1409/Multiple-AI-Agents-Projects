# PROFESSOR-NOTES.md — Self-Healing SQL Analytics Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| SQL joins/aggregates/subqueries | You must judge if generated SQL is *reasonable*, not just "runs" | Any standard SQL tutorial; skip if fluent |
| DB access control (SQLite `mode=ro`/`PRAGMA query_only`; Postgres `GRANT`) | The safety boundary depends on it | SQLite "URI filenames" + "PRAGMA" docs; Postgres "GRANT" |
| Benchmark methodology (metric definition, difficulty tags) | You report a Spider number precisely | Spider paper/README on its evaluation |
| `sqlglot` parsing basics | The static guard parses SQL into an AST | sqlglot README |

## 2. Core Concepts Taught

### Self-correction / bounded-retry loops
**What.** The agent checks its own output and, on failure, retries with the failure fed back — up to a fixed cap.
**Why it exists.** LLM-generated queries fail non-trivially often on the first try (wrong column, wrong join). A bounded loop recovers many the way a developer would: read the error, fix, retry.
**How it works (mechanism).** execute → check against cheap heuristics → on failure, put the *specific* error/violation into the next prompt (not "try again"). The specificity matters: the model needs the actual error ("no such column: cust_id") to correct, not a generic nudge. Stop after the budget regardless.
**Where here.** The Self-Check → SQL Generator loop, capped at 3.

### Schema-grounded generation (schema linking)
**What.** Give the LLM the real schema (tables, columns, types, FKs, sample rows) before asking for SQL.
**Why it exists.** Without grounding, the LLM invents plausible column names that don't exist. Grounding is what makes NL-to-SQL work beyond toys. The sub-problem — picking *which* tables to include when a DB has 50 — is "schema linking," itself an active research area; caching the full schema per DB is a fine middle ground here.
**Where here.** The Schema Inspector, cached per DB (a real cost/latency lesson).

### Benchmark-driven evaluation (Spider)
**What.** Evaluate against a standardized dataset with a defined metric, not ad-hoc questions.
**Why it exists.** "It worked on my 5 questions" isn't comparable or credible. Spider's **execution accuracy** (does running the query produce the gold result set) gives external validity. Note it's imperfect: two different questions can share a result set (false positives), and underspecified questions have multiple valid answers — so report the number honestly, not as ground truth.
**Where here.** The whole §6 eval.

### Least-privilege / structural safety
**What.** Make an unsafe action *impossible* given permissions, not merely *discouraged* by a prompt.
**Why it exists.** Prompts aren't a security boundary — an LLM can be manipulated or err into a destructive statement. The reliable defense removes the capability (read-only connection) and adds a pre-execution parser guard.
**Where here.** SQLite `mode=ro` + `PRAGMA query_only` + the `sqlglot` statement-type allowlist, with an adversarial DoD test.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Least-privilege DB access + AST-level guarding | A concrete, describable security practice |
| 1 | Schema-grounded NL-to-SQL | The baseline text-to-SQL skill on most data/AI JDs |
| 2 | Cheap rule-based checks before another LLM call | Cost/latency-aware engineering instinct |
| 3 | Tabular results → chart spec + summary | Any "AI analytics" feature |
| 4 | Running + honestly reporting a benchmark | The "one number beats adjectives" lesson |
| 5 | Deploy reinforcement | Repetition makes deployment second nature |

## 4. Common Misconceptions & Mistakes

- **"No error" = "correct."** Add the plausibility dimension.
- **Prompt-based safety.** Use the connection + guard; verify adversarially.
- **Regenerating schema per question.** Cache per connection.
- **Number without subset/metric.** Always footnote it.
- **Unbounded retries.** Hard cap in code.

## 5. Understanding-check questions (with answer key)

**Q1 (Self-correction).** Why is feeding the *specific* error better than "try again"? What happens with the latter?
**A1.** The specific error tells the model what to fix ("no such column: cust_id" → use `customer_id`), so the retry is targeted. "Try again" gives no new information; the model re-samples the same distribution and usually reproduces a similar error — non-learning retries.

**Q2 (Schema grounding).** What breaks with no schema? With every table of a 50-table DB in every prompt?
**A2.** No schema → the model invents non-existent columns/tables (hallucinated SQL). All 50 tables → wasted tokens, higher latency/cost, and *worse* accuracy (the model is distracted by irrelevant tables) — motivating schema linking.

**Q3 (Benchmark).** Why is execution accuracy better than exact string match against gold SQL?
**A3.** Semantically equivalent queries can be written many ways (different join order, aliases, subquery vs. JOIN). String match penalizes correct-but-different SQL; execution match compares result sets, which is what actually matters.

**Q4 (Least-privilege).** Design an adversarial test for a destructive-statement vulnerability. What and where do you check?
**A4.** Prompt the agent with something like "delete all rows where amount is 0" (or inject it), then assert: (a) the `sqlglot` guard classifies the generated statement as non-SELECT and rejects it; (b) even if bypassed, the read-only connection raises on the write. Check at both the guard (pre-exec) and the driver (exec), not just the LLM output.

**Q5 (Self-check debugging).** Self-healing rate is only 20%. Name three places the bug could be and how to isolate.
**A5.** (1) Feedback not actually in the retry prompt → log the prompt and confirm; (2) checks too weak (they pass wrong results, so no retry triggers) → inspect false "ok" verdicts; (3) generator can't use the feedback (schema wrong/missing) → check whether retried SQL differs and addresses the error. Isolate by logging, per failed question, whether a retry fired, whether SQL changed, and whether the change targeted the reported error.
