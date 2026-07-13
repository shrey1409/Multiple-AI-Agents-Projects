# PROFESSOR-NOTES.md — Agent Release Gate: CI/CD for Agents

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Project 03 (or its harness design) | This packages that pattern | Your own Project 03, or its PLAN.md |
| GitHub Actions (workflows, triggers, jobs, `actions/cache`) | The deliverable is an Action | GitHub Actions docs → "Understanding GitHub Actions" |
| Checks API / Commit Status API | The Action must pass/fail a PR check, not just print | GitHub "Checks API" / "Commit statuses" docs |
| `pull_request` vs `pull_request_target` + fork secrets | The real security footgun for a CI eval that needs a secret | GitHub "Security hardening for GitHub Actions" → events + secrets |
| Flaky tests / tolerance | Your regression check needs a band, not exact-threshold | Any "flaky tests in CI" write-up |

## 2. Core Concepts Taught

### CI/CD for non-deterministic systems
**What.** Applying CI discipline (checks on every change, blocking on regressions) to systems whose outputs vary run-to-run.
**Why it exists.** Traditional CI assumes same-input→same-output; LLM agents don't. A regression gate needs statistical aggregation (Project 03's scorecard) + tolerance bands, or it's useless (never fails) or unusable (fails randomly).
**How it works (mechanism).** Aggregate a success *rate* over many runs, compare to baseline with a significance test + tolerance band; only a change beyond normal variance blocks. This is Project 03's math, invoked per-PR.
**Where here.** The Action's comparison logic.

### Reusable infrastructure vs. project-specific tooling
**What.** A tool built to be installed/configured/used by *someone else's* project without editing its source, vs. one that solves your problem once.
**Why it exists.** "I built an eval harness for my agent" is far weaker than "I built a tool other teams install for theirs" — the second requires real API/config design discipline (what varies per-user must be config, not code).
**Where here.** Config-in-the-target-repo + the Phase-5 zero-source-change proof, analogous to Project 09's cross-client verification and Project 08's framework-swap.

### Baseline-relative comparison
**What.** Evaluate a candidate not against an absolute bar alone, but relative to the last known-good version's own measured performance.
**Why it exists.** Absolute thresholds are hard to set once (what's "good enough" success rate is domain-specific), while "don't get meaningfully worse than what's deployed" is robust and self-calibrating as the agent improves.
**Where here.** The Baseline Store + Comparison — what makes the gate a true regression detector, not a static bar.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Refactoring a prototype into an importable package | Unglamorous, common, valuable |
| 1 | GitHub Actions mechanics (triggers, config, status) | Widely applicable DevOps skill |
| 2 | Wiring a complex eval into a constrained CI env | Real ML/AI teams do exactly this |
| 3 | Statistical regression detection with tolerance | A useful gate vs. a flaky ignored one |
| 4 | Designing a stable reporting interface | The report *is* the product experience |
| 5 | Proving reusability by exercising it | The strongest, most concrete resume claim here |
| 6 | Install docs for a stranger | Completes the "devops product, not a demo" positioning |

## 4. Common Misconceptions & Mistakes

- **Action only works with your own agent.** Then it's project tooling with ceremony.
- **Exact-threshold, zero-tolerance blocking.** Flaky failures on unchanged code erode trust.
- **No first-run bootstrap.** The first PR has no baseline; crashing/auto-failing is a common avoidable bug.
- **Letting CI runtime/cost balloon.** A slow/expensive gate gets disabled.
- **Unstable comment format.** Reviewers can't build the glance-at-it habit.

## 5. Understanding-check questions (with answer key)

**Q1 (Non-deterministic CI).** Why would zero-tolerance exact-threshold blocking fail more often on unchanged code than a tolerance-banded one? What's varying?
**A1.** LLM sampling, judge variance, and tool/latency jitter make the success rate wobble run-to-run even with identical code. A zero-tolerance check treats any downward wobble as a regression and fails; a tolerance band only fails on a drop beyond normal variance.

**Q2 (Reusable infra).** What concrete test proves the Action is genuinely reusable, not "designed to be"? Why doesn't intention count?
**A2.** Protecting a second, different repo (Project 02) by config only, with a byte-identical Action-source diff. Intention doesn't count because untested "reusability" almost always hides a hardcoded assumption; only exercising it on a second case surfaces them.

**Q3 (Baseline-relative).** Why is "don't regress from last known-good" often more practical than a fixed absolute threshold? When is absolute better?
**A3.** "Don't regress" self-calibrates: as the agent improves, the bar rises automatically, and you avoid arguing about the "right" absolute number. Absolute is better when there's a hard external requirement (e.g., a safety metric that must never drop below a contractual floor regardless of history).

**Q4 (Bootstrap).** A repo's first PR has no baseline. What should happen and why is crashing/auto-failing wrong?
**A4.** The first run should pass and record its scorecard as the baseline, logging "no baseline; establishing one." Crashing blocks all work on a repo that did nothing wrong; auto-failing trains people to ignore or bypass the gate — both defeat adoption.

**Q5 (Second repo).** Pointing at Project 02 needs one small Action code change. What does that reveal, and what do you fix?
**A5.** It reveals a leaked assumption — something repo-specific lives in the Action instead of in `gate-config.yml` (a hardcoded endpoint, scenario path, or threshold). Fix by moving that variable into the config schema so the Action source stays untouched across repos.
