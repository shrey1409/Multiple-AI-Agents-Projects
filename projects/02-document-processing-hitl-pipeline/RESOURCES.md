# RESOURCES.md — Document-Processing Pipeline with Human-in-the-Loop

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed present.

## 1. What to clone / download

### LangGraph human-in-the-loop + extraction references
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Tutorials moved to `examples/` (the old `docs/docs/tutorials/` paths are gone). Verified current paths:
- Extraction-with-retries: `examples/extraction/retries.ipynb` ✅ — adapt the retry-on-low-confidence idea for Extraction.
- Hierarchical teams (read-only, sub-graph composition): `examples/multi_agent/hierarchical_agent_teams.ipynb` ✅.
- **`interrupt()` itself:** read the installed package, not a tutorial URL — `python -c "import langgraph; help(langgraph.types.interrupt)"`, and the "Human-in-the-loop" concept page on the live docs site. The calling convention (`Command(resume=...)`) has changed across releases; trust the installed version.

### MediSuite-AI-Agent (claims workflow — read-only design reference)
```bash
git clone --depth 1 https://github.com/ahmedmansour5/MediSuite-Ai-Agent.git /tmp/ref-medisuite   # ✅
```
Closest real-world analog: intake → document → validation → claim-decision. Study the workflow shape, don't import the code.

### legalai (contract/clause extraction — read-only)
```bash
git clone --depth 1 https://github.com/firica/legalai.git /tmp/ref-legalai   # ✅
```
Clause-extraction prompt patterns if you use "contract" as a doc type.

### Agno legal/document reference (read-only)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
```
⚠️ The old `cookbook/examples/agents/legal_consultant.py` is **deleted** (cookbook renumbered). For document/knowledge chunking patterns read `cookbook/07_knowledge/`. Pattern reference only; you're not using Agno as the framework here.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| LangGraph `interrupt()` (installed pkg + docs) | `langgraph.types.interrupt` | Reuse — the core mechanism, not swappable | 3 |
| LangGraph extraction-with-retries | `examples/extraction/retries.ipynb` | Adapt — retry-on-low-confidence for Extraction | 1 |
| LangGraph hierarchical teams | `examples/multi_agent/hierarchical_agent_teams.ipynb` | Read-only — sub-graph composition if you split Classify+Extract | 3 |
| MediSuite-AI-Agent | repo root | Read-only — intake→validate→decide shape | 1, 2 |
| legalai | repo root | Read-only — clause-extraction prompts | 1 |
| Agno knowledge cookbook | `cookbook/07_knowledge/` | Read-only — chunking/embedding patterns | 0 |

## 3. Data
- **Synthetic, not real.** Generate ~500 invoices/claims/contracts with an LLM that emits document + ground-truth JSON together (§5 of PLAN.md). Supplement with a few publicly redacted samples. Do **not** use real customer/PII data.

## 4. India-specific (localization)
- **No new repos — data stays synthetic.** Generate format-valid fake GST invoices, insurance claims, and KYC docs (PAN `[A-Z]{5}[0-9]{4}[A-Z]`, format-valid but fake Aadhaar with a correct **Verhoeff** check digit) + ground-truth JSON. **Never use real Aadhaar/PAN.**
- **GSTIN checksum:** GSTIN's 15th char is a checksum over the first 14 (mod-36) — implement it as a deterministic validation rule (a great "validate, don't pattern-match" example). Spec: search "GSTIN check digit algorithm" (GSTN documentation).
- **DPDP Act 2023 (learning reference):** read the Act's data-minimization + purpose-limitation principles (meity.gov.in) — the "why" behind masking Aadhaar to last-4 and treating the audit log as a processing record.
- **Regulators for context:** GSTN (tax), IRDAI (insurance), RBI (banking KYC).
