# PROFESSOR-NOTES.md — Agent Evaluation & Observability Platform

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Project 01 (or any target agent) already working | You need something to evaluate — this project is meaningless without a target | Your own Project 01 build |
| Basic statistics: precision/recall, agreement metrics (Cohen's κ) | You will report judge-vs-human calibration; "it looked right" is not a metric | Any short stats primer on inter-rater agreement |
| Prompt-injection basics (conceptual, defensive) | Your adversarial personas include injection attempts; you should understand what you're simulating | OWASP's LLM Top 10 entry on prompt injection |
| CI/CD basics (what a GitHub Action is, `on: pull_request` triggers) | Phase 5 wires the harness into a real CI gate | GitHub Actions' own quickstart docs |

## 2. Core Concepts Taught

### LLM-as-judge
**What:** using a (typically strong) LLM to score another system's output against a rubric, instead of relying only on exact-match metrics or human review.
**Why it exists:** for open-ended agent behavior (did it complete the task well, was the tone right, did it hallucinate), exact-match string comparison doesn't work, and human review doesn't scale to "run this on every commit." LLM-as-judge trades some accuracy for scale.
**How it works:** feed the judge the full transcript/trace plus explicit success criteria and a scoring rubric with structured output; critically, calibrate it against a hand-labeled sample so you know *how much* to trust it, rather than assuming it's correct.
**Where it's used here:** the LLM-as-Judge Evaluator node, and specifically the calibration step in Phase 3 — the part most student projects skip.

### Simulated-user evaluation
**What:** using an LLM to play the role of a user (with a defined persona/goal) and converse with your target agent automatically, instead of writing scripted test inputs by hand.
**Why it exists:** agents fail in multi-turn, context-dependent ways that single-shot test cases miss (the user changes their mind, gives contradictory info, or tries to manipulate the agent). A simulator can generate many such conversations cheaply and consistently.
**How it works:** a persona prompt defines the simulator's goal and behavior style; it converses turn-by-turn with the target, and the conversation continues until a max-turn limit or a natural stopping point.
**Where it's used here:** the User-Simulator agent, split into "scripted" (cooperative, tests the happy path) and "adversarial" (tests robustness) scenario kinds.

### Trajectory / trace analysis
**What:** treating an entire multi-turn interaction (every message, tool call, and result) as the unit of evaluation, not just the final answer.
**Why it exists:** an agent can reach a correct final answer through an incorrect process (e.g., calling the wrong tool but getting lucky, or hallucinating an intermediate fact that happens not to affect the final number) — trajectory-level evaluation catches process failures that outcome-only evaluation misses.
**Where it's used here:** the Trace Collector capturing every turn and tool call, and the Judge scoring `tool_call_correctness` as a separate dimension from `task_success`.

### Regression testing for non-deterministic systems
**What:** comparing a new version of a probabilistic system against a previous version's behavior to detect quality regressions, using statistical aggregates (since individual runs vary) rather than exact output comparison.
**Why it exists:** traditional software regression testing assumes determinism (same input → same output); LLM-based agents don't have that property, so you need aggregate metrics across many runs (success rate, not "did output X match exactly") to detect real regressions vs. normal variance.
**Where it's used here:** the Aggregator/Regression Detector comparing `VersionScorecard`s across target-agent versions, and the CI gate.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Scenario design for agent testing | Writing good test scenarios for non-deterministic systems is a distinct skill from writing unit tests |
| 1 (Simulator) | Building an LLM-driven test harness | Directly reusable for testing any future agent you build |
| 2 (Adversarial) | Adversarial thinking about your own systems | The mindset behind red-teaming and safety evaluation — increasingly a named job function |
| 3 (Judge) | Calibrating an automated grader against ground truth | This single skill — proving your eval is trustworthy, not just running one — is what makes this project rare |
| 4 (Regression) | Statistical thinking about non-deterministic regression detection | Distinguishes "I ran some tests" from "I built a system that would actually catch a real regression" |
| 5 (Dashboard+CI) | Shipping eval as an actual CI gate, not a notebook | This is the exact shape of "eval-driven development for agents" that's an emerging job requirement |
| 6 (Polish) | Telling a data-backed story about a regression you actually caught | The strongest possible resume artifact: a concrete before/after number, not an adjective |

## 4. Common Misconceptions & Mistakes

- **"The judge is basically ground truth."** It isn't — it's a model with its own biases and blind spots. Calibration against hand-labels is not optional polish; without it you cannot defend the judge's scores to a skeptical interviewer.
- **Vague rubrics.** "Rate the response 1-5 for quality" produces scores close to noise. Every dimension needs concrete anchors: what does a 1 look like vs. a 5, specifically.
- **Conflating outcome and process.** A trajectory that reaches the right answer via a wrong tool call (or by luck) is a different failure mode than one that fails outright — score them separately.
- **Building adversarial personas that are just "be mean."** Real adversarial testing needs concrete tactics (contradiction, scope creep, injection attempts), not a vague instruction to be difficult.
- **Skipping the "deliberately break the target and see if the harness notices" test.** Without this, you have no evidence your regression detector actually detects anything — it might just always say "no regression."

## Understanding-check questions

**After §2 (LLM-as-judge):** Why can't you just trust the judge's scores without calibration? What would a Cohen's κ of 0.3 tell you that a raw accuracy number wouldn't?

**After §2 (Simulated-user evaluation):** What kinds of agent failures show up only in multi-turn conversations and would be invisible to a single-shot test case?

**After §2 (Trajectory analysis):** Describe a scenario where an agent gets the final answer right but the trajectory should still be scored as a failure.

**After §2 (Regression testing):** Why do you need many runs and statistical aggregation to detect a regression in an LLM-based system, when traditional software tests compare one output to one expected value?

**After Phase 4 (Regression detection):** You inject 3 broken versions and the harness only catches 2. How would you debug whether the problem is the judge's rubric, the scenario coverage, or the aggregation threshold?
