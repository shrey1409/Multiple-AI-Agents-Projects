# RESOURCES.md — Agent Guardrail Gateway

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED · ❓ UNCONFIRMED (flagged, do not trust blindly).

## 1. What to clone / download

### NeMo Guardrails (NVIDIA) — read-only design reference

```bash
git clone --depth 1 https://github.com/NVIDIA/NeMo-Guardrails.git /tmp/ref-nemo-guardrails
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). A production-grade guardrails framework — read its docs for how it structures input/output rails and action allow-listing; useful design-pattern reference even if you hand-roll your own gateway rather than adopting this framework wholesale (hand-rolling is recommended here so you actually learn the mechanics, per this project's PLAN.md rationale).

### Guardrails AI (guardrails-ai/guardrails) — read-only design reference

```bash
git clone --depth 1 https://github.com/guardrails-ai/guardrails.git /tmp/ref-guardrails-ai
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). Another established guardrails library, structured around validators — read for its validator-composition pattern as a second reference point alongside NeMo Guardrails.

### Microsoft Presidio (PII detection/anonymization) — ❓ location unconfirmed

```bash
# The exact current canonical GitHub location for Presidio could not be confirmed during
# this planning session (raw README checks against microsoft/presidio and a
# possible data-privacy-stack/presidio successor both 404'd). Search for
# "Microsoft Presidio PII detection" at implementation time to find the current repo
# rather than trusting either URL below without verifying it first:
# https://github.com/microsoft/presidio  (may have moved/transferred stewardship)
```
**❓ UNCONFIRMED — verify before relying on this.** If Presidio is genuinely unavailable or moved, spaCy's built-in NER (`en_core_web_sm` or similar) plus a regex library for structured PII (SSNs, credit-card-like numbers, phone numbers) is a perfectly adequate substitute for Phase 1's PII redaction layer and doesn't depend on this specific project.

### OWASP LLM Top 10 project (prompt-injection reference)

```bash
git clone --depth 1 https://github.com/OWASP/www-project-top-10-for-large-language-model-applications.git /tmp/ref-owasp-llm-top10
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). This is your primary source for building a defensible, documented prompt-injection pattern library (PLAN.md §8) rather than inventing attack patterns ad hoc.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| NeMo Guardrails | Read-only — rail/action-allow-list design pattern reference | Phase 2, 4 |
| Guardrails AI | Read-only — validator composition pattern reference | Phase 1, 2 |
| Presidio (if located) or spaCy NER fallback | Reuse as-is (Presidio) or adapt (spaCy + regex) — your PII detection engine | Phase 1 |
| OWASP LLM Top 10 | Reuse as-is — source patterns for your adversarial prompt-injection test set | Phase 2, 5 |

## 3. Target agent

This project requires a target agent to protect — reuse Project 01's FastAPI endpoint (built in this same portfolio) rather than building a new one, keeping the gateway's own build focused on the 4 protections rather than re-solving agent design.
