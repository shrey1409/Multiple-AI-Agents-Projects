# PROFESSOR-NOTES.md — Agent Evaluation & Observability Platform

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| A working target agent (Project 01/02) | Nothing to evaluate without one | Your own build |
| Precision/recall + inter-rater agreement (Cohen's κ) | You report judge-vs-human calibration | Any stats primer on κ; know the formula below |
| Prompt injection (defensive, conceptual) | Your adversarial personas simulate it | OWASP LLM Top 10 → LLM01 Prompt Injection |
| GitHub Actions (`on: pull_request`) | Phase 5 wires a CI gate | GitHub Actions "Understanding GitHub Actions" |

## 2. Core Concepts Taught

### LLM-as-judge
**What.** Use a strong LLM to score another system's output against a rubric instead of exact-match or human-only review.
**Why it exists.** For open-ended agent behavior (task done well? tone right? hallucinated?), string comparison fails and human review doesn't scale to every commit. LLM-as-judge trades some accuracy for scale.
**How it works (mechanism + its failure).** Feed the judge the trajectory + explicit criteria + a rubric with structured output. The catch: the judge has systematic biases — **position bias** (prefers the first of two options), **verbosity bias** (prefers longer answers), **self-enhancement bias** (prefers outputs from its own model family). You *measure* how much to trust it via calibration; you don't assume it.
**Where here.** The Judge node + the Phase-3 calibration most projects skip.

### Cohen's κ (why not raw agreement)
**What.** A chance-corrected agreement statistic: `κ = (p_o − p_e) / (1 − p_e)`, where `p_o` is observed agreement and `p_e` is agreement expected by chance from the label marginals.
**Why it exists.** On imbalanced labels, raw agreement lies: if 90% of trajectories "succeed," a judge that always says "success" scores 90% agreement while being useless. κ subtracts the chance baseline, so that judge scores ≈0.
**Where here.** The ≥0.6 calibration target — 0.6 is "substantial" agreement; below that the judge isn't trustworthy yet.

### Simulated-user evaluation
**What.** An LLM plays a user with a persona/goal and converses with the target automatically.
**Why it exists.** Agents fail in multi-turn, context-dependent ways single-shot tests miss (user changes mind, contradicts, manipulates). A simulator generates many such conversations cheaply.
**Where here.** The User-Simulator, split scripted (happy path) vs. adversarial (robustness).

### Trajectory / trace analysis
**What.** Evaluate the whole interaction (every message, tool call, result), not just the final answer.
**Why it exists.** An agent can reach a right answer via a wrong process (lucky wrong tool call, harmless intermediate hallucination). Trajectory-level scoring catches process failures outcome-only scoring misses.
**Where here.** The Trace Collector + scoring `tool_call_correctness` separately from `task_success`.

### Regression testing for non-deterministic systems
**What.** Compare a new version to a previous one using statistical aggregates, since individual runs vary.
**Why it exists.** Traditional regression testing assumes determinism; LLM agents don't have it. You need aggregate metrics (success *rate* over many runs) plus a significance test, or you flag noise.
**Where here.** The Aggregator's two-proportion test + Wilson CI.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Scenario design for non-deterministic systems | Distinct skill from unit tests |
| 1 | Building an LLM-driven test harness | Reusable for any future agent |
| 2 | Adversarial thinking about your own systems | The red-teaming/safety-eval mindset, a named job function |
| 3 | Calibrating a grader against ground truth | Proving your eval is trustworthy is what makes this project rare |
| 4 | Statistical regression detection | "I built a system that would catch a real regression" |
| 5 | Eval-as-CI, not a notebook | Eval-driven development for agents, an emerging requirement |
| 6 | A data-backed regression story | A concrete before/after number beats adjectives |

## 4. Common Misconceptions & Mistakes

- **"The judge is ground truth."** It's a biased model; calibrate or you can't defend its scores.
- **Vague rubrics.** Anchor every dimension.
- **Outcome/process conflation.** Score them separately.
- **"Be mean" adversarial personas.** Use concrete tactics.
- **Skipping the deliberately-break-the-target test.** Without it you have no evidence the detector detects anything.
- **Bare-threshold gating at small n.** Flags noise; use the significance test.

## 5. Understanding-check questions (with answer key)

**Q1 (LLM-as-judge).** Why can't you trust judge scores without calibration? What does κ=0.3 tell you that raw accuracy wouldn't?
**A1.** The judge has biases and blind spots; without calibration you don't know its error rate. κ=0.3 ("fair") means most of the raw agreement is explainable by chance — the judge barely beats guessing — which a high raw-accuracy number on imbalanced labels would hide.

**Q2 (Simulated users).** What failures show up only in multi-turn and are invisible to single-shot tests?
**A2.** Context drift (agent forgets an earlier stated constraint), inconsistency across turns, failure to handle a user changing their mind, and susceptibility to a multi-turn manipulation that builds up.

**Q3 (Trajectory).** Describe a case where the final answer is right but the trajectory should score as failure.
**A3.** The agent calls the wrong tool (e.g., queries the news tool for a price), gets a plausible-but-lucky number that happens to match, and answers correctly. Outcome = pass, but the process is broken and will fail on the next input.

**Q4 (Regression).** Why do you need many runs + aggregation to detect a regression, when traditional tests compare one output to one expected value?
**A4.** LLM output varies run-to-run, so a single differing output isn't a regression — it's noise. Only the *rate* over many runs, tested for significance, distinguishes a real quality drop from variance.

**Q5 (Regression detection).** You inject 3 broken versions and catch only 2. How do you debug whether it's the rubric, coverage, or threshold?
**A5.** Check the missed version's judgments: if the judge scored the broken trajectory as success → rubric/coverage gap (no scenario exercises the broken behavior, or the rubric doesn't penalize it). If judgments show the drop but `regression_flags` didn't fire → threshold/statistical-test problem (tolerance too wide or n too small for significance).
