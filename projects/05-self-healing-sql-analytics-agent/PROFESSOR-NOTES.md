# PROFESSOR-NOTES.md — Self-Healing SQL Analytics Agent

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| SQL fundamentals (joins, aggregates, subqueries) | You need to judge whether generated SQL is *reasonable*, not just "runs" | Any standard SQL tutorial — skip if you already know SQL |
| Database roles/permissions (`GRANT`, read-only users) | The safety boundary in this project depends on it | Your DB's own docs on role-based access (Postgres or SQLite-adjacent tooling) |
| What a benchmark is and why methodology matters (train/test split, evaluation metric definition) | You'll report a Spider-benchmark number and need to state it precisely | Spider benchmark's own paper/README on its evaluation methodology |

## 2. Core Concepts Taught

### Self-correction / bounded-retry loops
**What:** an agent that checks its own output, and — if the check fails — retries with the failure fed back into the next attempt, up to a fixed number of times.
**Why it exists:** LLM-generated code/queries fail non-trivially often on the first try (wrong column name, wrong join, syntax slip); rather than surfacing every first-attempt failure to the user, a bounded self-correction loop recovers many of them automatically, closing the loop the same way a human developer would (read the error, fix it, try again).
**How it works:** execute → check the result against cheap heuristics (did it error, is it empty, are values absurd) → if it fails, feed the specific error/heuristic violation back into the generator's next prompt, not just "try again" — and stop after a fixed retry budget regardless of outcome.
**Where it's used here:** the Self-Check → SQL Generator retry loop, bounded at 3 attempts.

### Schema-grounded generation (schema linking)
**What:** giving an LLM the actual database schema (table/column names, types, relationships, sample rows) as context before asking it to generate a query against that specific database.
**Why it exists:** without schema grounding, an LLM will confidently invent plausible-sounding column names that don't exist in your actual database — schema grounding is what makes NL-to-SQL work at all beyond toy examples.
**Where it's used here:** the Schema Inspector step, cached once per database rather than regenerated per question (a real cost/latency lesson, not just a nicety).

### Benchmark-driven evaluation (Spider)
**What:** evaluating your system against a standardized, published benchmark dataset with a defined metric, instead of only your own ad-hoc test questions.
**Why it exists:** "it worked on the 5 questions I tried" is not comparable across projects or credible to a reviewer; a named benchmark with a known difficulty level and a precise metric definition (execution accuracy, here) gives your resume claim external validity.
**Where it's used here:** the entire eval strategy in PLAN.md §6 — this project's core differentiator versus just building "a NL-to-SQL demo."

### Least-privilege / structural safety boundaries
**What:** designing a system so that an unsafe action is *impossible* given the permissions it has, rather than merely *discouraged* by instructions.
**Why it exists:** prompts are not a security boundary — an LLM can be manipulated or simply err into generating a destructive statement; the only reliable defense is removing its capability to do damage at the infrastructure layer (a read-only DB role here).
**Where it's used here:** the read-only DB connection, and the explicit adversarial test in the Definition of Done that proves the boundary actually holds.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Configuring least-privilege DB access | A concrete, checkable security practice you can describe in an interview |
| 1 (Happy path) | Schema-grounded NL-to-SQL generation | The baseline "text-to-SQL agent" skill listed in most data/AI-engineer job postings |
| 2 (Self-check+retry) | Designing cheap, rule-based sanity checks before reaching for another LLM call | Teaches judgment about when a heuristic beats another model call — a cost/latency-aware engineering instinct |
| 3 (Narrative+chart) | Turning tabular results into a structured chart spec + natural-language summary | Directly applicable to any "AI analytics" product feature |
| 4 (Benchmark eval) | Running and honestly reporting a standardized benchmark | Distinguishes your project from unverifiable claims — the "one number beats adjectives" lesson |
| 5 (Deploy) | Same deploy skills as other projects, reinforced | Repetition across the portfolio is intentional — deployment should become second nature |

## 4. Common Misconceptions & Mistakes

- **"No error" is treated as "correct."** A query that runs cleanly can still answer the wrong question; self-checks need more than an error/no-error signal.
- **Relying on prompt instructions for safety instead of DB permissions.** This is the single most common security mistake in NL-to-SQL demos — always verify the boundary with an actual adversarial test, don't just trust the system prompt.
- **Regenerating the full schema context on every question.** Wastes tokens and latency; cache it per connection/session.
- **Reporting an accuracy number without specifying the subset and metric.** "70% on Spider" needs a footnote: which databases, how many questions, execution accuracy vs. exact match.
- **Unbounded retries "until it works."** Without a hard cap, a genuinely unanswerable or malformed question can loop indefinitely and blow your latency/cost budget.

## Understanding-check questions

**After §2 (Self-correction):** Why is feeding the *specific* execution error back into the next generation attempt better than simply re-prompting "try again"? What would you expect to happen with the latter?

**After §2 (Schema-grounded generation):** What goes wrong if you don't give the LLM the actual schema at all? What goes wrong if you give it every table in a 50-table database for every question?

**After §2 (Benchmark evaluation):** Why is execution accuracy (does running the query give the right result set) a better metric here than exact string match against the gold SQL?

**After §2 (Least-privilege):** Design an adversarial test that would catch a destructive-statement vulnerability in this project. What exactly would you check, and where (at the LLM output, or at execution time)?

**After Phase 2 (Self-check+retry):** Your self-healing rate is only 20% — most retries still fail. What are three different places in the pipeline the bug could actually be, and how would you isolate which one it is?
