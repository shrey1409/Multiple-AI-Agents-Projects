# RESOURCES.md — Computer-Use / Browser Automation Agent

Paths verified 2026-07-13 (HTTP 200). ✅ = confirmed.

## 1. What to clone / read

### Playwright (Python) — the automation library
```bash
git clone --depth 1 https://github.com/microsoft/playwright-python.git /tmp/ref-playwright-python   # ✅
pip install playwright
# DO NOT run `playwright install` in this environment — Chromium is pre-installed at
# /opt/pw-browsers (PLAYWRIGHT_BROWSERS_PATH). Launch with executablePath if a version pins differently.
```
Your driver. Read the accessibility snapshot API (`page.accessibility.snapshot()`) and role-based locators — the a11y-first grounding path.

### browser-use — agent-driving-browser design reference (read-only)
```bash
git clone --depth 1 https://github.com/browser-use/browser-use.git /tmp/ref-browser-use   # ✅
```
A popular "LLM controls a browser" library. Read how it exposes the page to the model (DOM/a11y serialization + element indexing) and structures the action loop — strong prior art for your Observer + Grounder. Build your own typed/gated version rather than adopting wholesale.

### WebArena / VisualWebArena — benchmark task design (read-only)
```bash
git clone --depth 1 https://github.com/web-arena-x/webarena.git       /tmp/ref-webarena         # ✅
git clone --depth 1 https://github.com/web-arena-x/visualwebarena.git /tmp/ref-visualwebarena   # ✅
```
Read for **task formats and success-assertion patterns** (how they define "task complete" on self-hostable sites). Use their task *design*; score against your own controlled/sandbox sites for reproducibility and safety. VisualWebArena specifically covers visually-grounded tasks relevant to your vision fallback.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| Playwright Python | repo / `pip install playwright` | Reuse — browser driver + a11y snapshots | 0, 1, 3 |
| browser-use | repo | Read-only — page-serialization + action-loop design | 1, 2 |
| WebArena | repo | Read-only — task format + success assertions | 5 |
| VisualWebArena | repo | Read-only — visually-grounded task design (vision fallback) | 3, 5 |

## 3. Benchmark sites (data)
Self-host deterministic targets for scoring: a sandbox e-commerce/test shop, form-filling pages, a to-do web app (WebArena ships self-hostable site images you can reuse). Score against these — never against live public sites (non-deterministic, ToS/rate-limit risk, real transactions).

## 4. Reused across the portfolio
- The action trace doubles as the Target Agent Contract trajectory (Projects 03/13 can observe this agent).
- The irreversible-action gate reuses Project 04's structural-approval pattern.
