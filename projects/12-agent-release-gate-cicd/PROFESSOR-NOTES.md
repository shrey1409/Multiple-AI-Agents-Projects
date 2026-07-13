# PROFESSOR-NOTES.md — Agent Release Gate: CI/CD for Agents

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Project 03 (or at least its eval-harness design) | This project packages that exact pattern for reuse | Your own Project 03 build, or this portfolio's Project 03 PLAN.md if you're sequencing differently |
| GitHub Actions fundamentals (workflows, triggers, jobs, steps, the `actions/cache` action) | The entire deliverable is a GitHub Action | GitHub's own Actions documentation, starting with "Understanding GitHub Actions" |
| What a CI "check" and a "status" API call are | Your Action needs to fail/pass a PR's checks, not just print output | GitHub's Checks API / Commit Status API docs |
| Basic idea of statistical tolerance / flakiness in testing | Your regression comparison needs a tolerance band, not exact-threshold blocking | Any short discussion of "flaky tests" in a CI/CD context (a very well-known software-engineering problem, not agent-specific) |

## 2. Core Concepts Taught

### CI/CD for non-deterministic systems
**What:** applying continuous-integration discipline (automated checks on every code change, blocking merges on regressions) to systems whose outputs vary run-to-run, which breaks the usual assumption that a test either deterministically passes or fails.
**Why it exists:** traditional CI assumes "same input, same output" tests; LLM-based agents don't have that property, so a regression gate needs statistical aggregation (Project 03's `VersionScorecard`) and tolerance bands rather than exact comparisons — otherwise the gate is either useless (never fails) or unusable (fails randomly on unchanged code).
**Where it's used here:** the entire Action's comparison logic, and specifically the tolerance-band design note in PLAN.md §8.

### Reusable infrastructure vs. project-specific tooling
**What:** the distinction between building a tool that solves your problem once, and building a tool designed from the start to be installed, configured, and used by someone else's different project without modifying its source.
**Why it exists:** "I built an eval harness for my agent" is a much weaker engineering claim than "I built a tool other teams can install for their agents" — the second requires genuine API/config design discipline (what varies per-user must be config, not code) that the first never forces you to practice.
**Where it's used here:** the config-file-lives-in-the-target-repo design, and the Phase 5 requirement to protect a second target repo with zero Action source changes — the single most important validation in this project, directly analogous to Project 09's cross-client verification and Project 08's framework-swap proof.

### Baseline-relative comparison
**What:** evaluating a candidate (a PR's version of an agent) not against an absolute pass/fail bar alone, but relative to the last known-good version's own measured performance.
**Why it exists:** absolute thresholds are hard to set correctly once (what counts as "good enough" success rate is genuinely domain-specific), while "don't get meaningfully worse than what's already deployed" is a robust, self-calibrating standard that adapts as the agent improves over time.
**Where it's used here:** the Baseline Store and Comparison step — this is what makes the gate a true regression detector, not just a static quality bar.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Refactoring a working prototype into an importable, reusable package | An unglamorous but extremely common and valuable real-world engineering task |
| 1 (Action skeleton) | GitHub Actions mechanics (triggers, config reading, status reporting) | A widely applicable DevOps skill, not agent-specific, that pairs well with any agent-engineering role |
| 2 (Harness integration) | Wiring a complex evaluation pipeline into a constrained CI environment | Real production ML/AI teams do exactly this for model and agent regression testing |
| 3 (Baseline+Comparison) | Statistical regression detection with tolerance bands | Distinguishes a genuinely useful CI gate from a flaky, ignored one |
| 4 (Scorecard comment) | Designing a stable reporting interface/template | A product-thinking skill — the report *is* the product experience for whoever reviews the PR |
| 5 (Second target repo) | Proving reusability by actually exercising it on a second, independent case | The single strongest, most concrete claim in this project's resume bullet |
| 6 (Polish) | Writing installation docs for a tool a stranger will actually adopt | Completes the "devops product, not a demo" positioning this project is built around |

## 4. Common Misconceptions & Mistakes

- **Building the Action to only work with your own agent(s).** If the config can't fully describe how to reach and evaluate a *different* repo's agent, you've built project-specific tooling with extra ceremony, not a reusable Action.
- **Exact-threshold, zero-tolerance regression blocking.** Given real LLM output variance, this produces flaky failures on unchanged code and quickly erodes trust in the gate — a tolerance band is a deliberate design choice, not a corner cut.
- **No sensible first-run/bootstrap behavior.** A repo's very first gate run has no baseline to compare against; crashing or flagging everything as a regression on that first run is a common, easily-avoidable bug.
- **Letting CI runtime/cost balloon unnoticed.** An eval gate too slow or expensive to run on every PR will get disabled — treat runtime and cost as first-class metrics, not afterthoughts.
- **Unstable comment/report formatting.** If the scorecard's shape changes every run, reviewers can't build the habit of glancing at it — treat the template as a versioned interface.

## Understanding-check questions

**After §2 (CI/CD for non-deterministic systems):** Why would a zero-tolerance, exact-threshold regression check likely fail more often on completely unchanged code than a tolerance-banded one? What's actually varying between runs?

**After §2 (Reusable infrastructure):** What's the concrete test that proves this Action is genuinely reusable, rather than just "designed to be" reusable? Why does intention not count as proof here?

**After §2 (Baseline-relative comparison):** Why is "don't regress from the last known-good version" often a more practical standard than a fixed absolute threshold? Can you think of a case where an absolute threshold would actually be preferable?

**After Phase 3 (Baseline+Comparison):** A repo's very first PR through this gate has no prior baseline. What should happen, and why is crashing or auto-failing the wrong behavior?

**After Phase 5 (Second target repo):** You point the Action at Project 02 and it needs one small code change to work. What does that tell you about your Phase 1-4 design, and what would you go back and fix?
