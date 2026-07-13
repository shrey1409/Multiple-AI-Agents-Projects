# RESOURCES.md — Agent Release Gate: CI/CD for Agents

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### Official GitHub Actions toolkit repos
```bash
git clone --depth 1 https://github.com/actions/toolkit.git        /tmp/ref-actions-toolkit         # ✅
git clone --depth 1 https://github.com/actions/cache.git          /tmp/ref-actions-cache           # ✅
git clone --depth 1 https://github.com/actions/github-script.git  /tmp/ref-actions-github-script   # ✅
```
- `toolkit` — SDK for a JS/TS action (a Docker/composite Python action is arguably simpler given this portfolio's Python stack; pick by comfort).
- `cache` — persist the baseline scorecard between runs (PLAN §3), keyed by `(repo, scenario_set_hash)`.
- `github-script` — post the scorecard PR comment (PLAN §4) without hand-rolling GitHub API auth.

### Your own Project 03 (primary internal dependency)
```bash
# No external clone — this is the harness you're packaging.
# Refactor its simulator + judge + aggregator into an importable `agent_eval_gate`
# package rather than copy-pasting between Project 03 and Project 12.
```

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| `actions/toolkit` | repo root | Read-only (or reuse for a JS action) | 1 |
| `actions/cache` | repo root | Reuse — baseline persistence | 3 |
| `actions/github-script` | repo root | Reuse — PR comment posting | 4 |
| Project 03 harness | (internal) | Adapt — refactor into an importable package | 0, 2 |

## 3. GitHub docs (load-bearing, non-code)
- "Creating a Docker container action" (natural fit for the Python harness — avoids re-installing deps per run) vs. "Creating a composite action".
- Checks API / Commit Status API — to actually block a PR's merge, not just comment.
- **"Security hardening for GitHub Actions" → events + secrets** — read the `pull_request` vs `pull_request_target` and fork-secret-exposure section before Phase 1; it's the real footgun for a CI eval that needs an LLM API key (see PLAN §8).

## 4. Target repos for the reusability proof
Reuse this portfolio's Project 01 and 02 as the two targets — their `gate-config.yml` files are the concrete artifacts proving the cross-repo reusability claim. Both already expose the Target Agent Contract endpoint the harness calls.
