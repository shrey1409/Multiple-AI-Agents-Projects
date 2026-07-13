# RESOURCES.md — Agent Guardrail Gateway

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed present.

## 1. What to clone / download

### NeMo Guardrails (NVIDIA) — read-only design reference
```bash
git clone --depth 1 https://github.com/NVIDIA/NeMo-Guardrails.git /tmp/ref-nemo-guardrails   # ✅
```
Production guardrails framework — read how it structures input/output rails + action allow-listing. Hand-roll your own gateway (so you learn the mechanics) but borrow the rail design.

### Guardrails AI — read-only design reference
```bash
git clone --depth 1 https://github.com/guardrails-ai/guardrails.git /tmp/ref-guardrails-ai   # ✅
```
Validator-composition pattern as a second reference alongside NeMo.

### Microsoft Presidio (PII detection) — CONFIRMED (corrects the earlier ❓)
```bash
git clone --depth 1 https://github.com/microsoft/presidio.git /tmp/ref-presidio   # ✅ EXISTS
```
**Correction:** the earlier file flagged Presidio as ❓ UNCONFIRMED — that was a planning-sandbox artifact. `microsoft/presidio` exists and is the recommended PII detection/anonymization engine. Use it for Phase 1, or fall back to spaCy `en_core_web_sm` NER + regex (with a Luhn check for card numbers) if you want zero extra heavy deps.

### OWASP LLM Top 10 (injection pattern source)
```bash
git clone --depth 1 https://github.com/OWASP/www-project-top-10-for-large-language-model-applications.git /tmp/ref-owasp-llm-top10   # ✅
```
Your primary source for a defensible, documented injection pattern library and the **named** adversarial corpus your ≥85% claim is scoped to.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| NeMo Guardrails | repo root | Read-only — rail/allow-list design | 2, 4 |
| Guardrails AI | repo root | Read-only — validator composition | 1, 2 |
| Presidio | `microsoft/presidio` | Reuse — PII engine (or spaCy+regex fallback) | 1 |
| OWASP LLM Top 10 | repo root | Reuse — injection patterns + named corpus | 2, 5 |

## 3. Target agent
Reuse Project 01's Contract-compliant endpoint as the protected target. The allow-list reads proposed actions from the target's returned `trajectory` (Target Agent Contract) — build Project 01 (or 02) first, or stub a Contract-shaped endpoint.

## 4. India-specific PII (localization)
Add Indian PII detectors alongside the US patterns (worldwide mode keeps both):
- **Aadhaar** — 12 digits with a **Verhoeff** checksum (the last digit validates the first 11). Implement Verhoeff (standard algorithm; search "Verhoeff algorithm Aadhaar validation") — a stronger deterministic-validation lesson than Luhn. **Mask to last-4.**
- **PAN** `^[A-Z]{5}[0-9]{4}[A-Z]$`, **UPI VPA** `^[\w.\-]+@[\w]+$`, **Indian mobile** `^(\+91|0)?[6-9]\d{9}$`, **IFSC** `^[A-Z]{4}0[A-Z0-9]{6}$`, **GSTIN** (15-char with checksum).
- **NER for Indian names/places:** spaCy multilingual, or **AI4Bharat IndicNER** (github.com/AI4Bharat — org verified). `indic-transliteration` (PyPI, verified) helps normalize scripts.
- **DPDP Act 2023** — the regulatory "why"; the gateway is a data-processor control (data minimization, purpose limitation).
