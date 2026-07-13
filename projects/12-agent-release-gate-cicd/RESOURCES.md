# RESOURCES.md — Agent Release Gate: CI/CD for Agents

Checked 2026-07-13. ✅ CONFIRMED WORKING.

## 1. What to clone / download

### Official GitHub Actions toolkit repos

```bash
git clone --depth 1 https://github.com/actions/toolkit.git /tmp/ref-actions-toolkit
git clone --depth 1 https://github.com/actions/cache.git /tmp/ref-actions-cache
git clone --depth 1 https://github.com/actions/github-script.git /tmp/ref-actions-github-script
```
Status: ✅ CONFIRMED WORKING (all three returned HTTP 200). `toolkit` is the SDK for building a custom JavaScript/TypeScript Action if you go that route (a Docker-based or composite Action in Python is also a valid, arguably simpler choice given this portfolio's Python-heavy stack — pick based on your comfort level, not because one is "more correct"). `cache` is the official caching action for persisting the baseline scorecard between runs (PLAN.md §3). `github-script` is a convenient way to post the PR scorecard comment (PLAN.md §4) without hand-rolling GitHub API auth boilerplate.

### Your own Project 03 (primary internal dependency)

```bash
# No external clone needed — this is the eval harness you're packaging.
# Refactor its simulator + judge + aggregator into an importable package
# (e.g. a small `agent_eval_gate` Python package) rather than copy-pasting
# code between Project 03 and Project 12.
```

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| `actions/toolkit` | Read-only reference (or reuse as-is if building a JS/TS Action) | Phase 1 |
| `actions/cache` | Reuse as-is — baseline scorecard persistence | Phase 3 |
| `actions/github-script` | Reuse as-is — PR comment posting | Phase 4 |
| Project 03's harness | Adapt — refactor into an importable package; this *is* the core of this project's Phase 0/2 | Phase 0, 2 |

## 3. GitHub's own documentation (not repos, but load-bearing)

- GitHub Actions "Creating a composite action" and "Creating a Docker container action" guides — pick whichever action type matches your harness's packaging (a Docker action is a natural fit if your harness has Python dependencies, since it avoids re-installing them on every runner).
- GitHub's Checks API / Commit Status API docs — needed to make the Action actually block a PR's merge (not just post an informational comment) when a regression is detected.

## 4. Target repos for the reusability proof

Reuse this same portfolio's Project 01 and Project 02 as the two target repos required by PLAN.md's success criteria — no external repos needed. This also means their `gate-config.yml` files are the concrete, checkable artifacts that prove the cross-repo reusability claim.
