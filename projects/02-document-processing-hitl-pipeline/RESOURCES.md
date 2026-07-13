# RESOURCES.md — Document-Processing Pipeline with Human-in-the-Loop

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### MediSuite-AI-Agent (claims workflow — read-only design reference)

```bash
git clone --depth 1 https://github.com/ahmedmansour5/MediSuite-Ai-Agent.git /tmp/ref-medisuite
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). Read this for the intake → document → validation → claim-decision workflow shape; the repo is a hospital/insurance claim automation, closest real-world analog to this project's domain.

### legalai (Legal Document Review Assistant — read-only for extraction/clause patterns)

```bash
git clone --depth 1 https://github.com/firica/legalai.git /tmp/ref-legalai
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). Useful for clause-extraction prompt patterns if you use "contract" as one of your three document types.

### Agno Legal Document Analysis Agent (vector-embedding legal PDF analysis)

```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
find /tmp/ref-agno/cookbook -iname "*legal*"
```
**⚠️ STALE/RESTRUCTURED.** `cookbook/examples/agents/legal_consultant.py` (curated link) 404s on `main` — Agno's cookbook was renumbered (see Project 01's RESOURCES.md for the full folder list). Run the `find` above; the legal-consultant example most likely now sits under `02_agents/` or `07_knowledge/` (knowledge/RAG-focused folder).

### LangGraph Extraction-with-Retries and Hierarchical Agent Teams tutorials

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
find /tmp/ref-langgraph -iname "*retries*" -o -iname "*extraction*"
find /tmp/ref-langgraph -iname "*hierarchical*"
```
**⚠️ STALE/RESTRUCTURED.** Both curated paths (`docs/docs/tutorials/extraction/retries.ipynb`, `docs/docs/tutorials/multi_agent/hierarchical_agent_teams.ipynb`) 404 on `main` as of 2026-07-13 — same docs reorganization noted in Project 01. Clone-and-`find` locally; you have normal network access at implementation time even though this planning session's proxy blocked the GitHub API needed to enumerate the new structure remotely.

### LangGraph's own `interrupt()` / human-in-the-loop concept docs

```bash
# Search the cloned repo for the current human-in-the-loop guide rather than guessing a URL:
grep -rl "def interrupt" /tmp/ref-langgraph/libs/langgraph/langgraph || true
find /tmp/ref-langgraph/docs -iname "*human*" -o -iname "*interrupt*"
```
This is the most important reference in this project — read the actual `interrupt` source/signature in the installed package version, since the API has changed across LangGraph releases; don't assume any specific tutorial's exact calling convention is current.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| MediSuite-AI-Agent | Read-only — study the intake→validate→decide shape; don't import its code (different stack/domain specifics) | Phase 1, 2 |
| legalai | Read-only — clause-extraction prompt ideas if "contract" is one of your doc types | Phase 1 |
| Agno Legal Document Analysis Agent (once located) | Read-only — see how they chunk/embed legal PDFs; you're not using Agno as your framework here | Phase 0 |
| LangGraph Extraction-with-Retries (once located) | Adapt — reuse the retry-on-low-confidence idea for your Extraction agent | Phase 1 |
| LangGraph Hierarchical Agent Teams (once located) | Read-only — this project is a linear pipeline, not a hierarchical team, but the tutorial's sub-graph composition pattern is useful if you later want to split Classify+Extract into their own sub-graph | Phase 3 |
| LangGraph `interrupt()` source/docs | Reuse the exact API as your primary mechanism — this is not optional or swappable | Phase 3 |

## 3. Data resources

- No agent-list link here — this phase needs **synthetic data**, not more agent code. Generate ~500 synthetic invoices/claims/contracts with an LLM (prompt it to also emit the ground-truth field values as JSON alongside each document) so you have exact accuracy scoring, and supplement with a small number of publicly available redacted samples for realism. Do not use real customer/PII data for this project.
