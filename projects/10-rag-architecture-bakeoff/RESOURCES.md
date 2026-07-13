# RESOURCES.md — RAG Architecture Bake-Off

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### RAGAS (evaluation library)

```bash
git clone --depth 1 https://github.com/explodinggradients/ragas.git /tmp/ref-ragas
pip install ragas
```
Status: ✅ CONFIRMED WORKING. This is the primary scoring library for the entire bake-off (§6 of PLAN.md) — read its docs on the exact current API for faithfulness/answer-relevancy/context-precision/context-recall, since the library's API surface changes across versions; don't assume any older tutorial's exact function calls are current.

### LangGraph's four RAG tutorials (Agentic, Adaptive, Corrective, Self-RAG)

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
find /tmp/ref-langgraph -iname "*agentic_rag*" -o -iname "*adaptive_rag*" -o -iname "*crag*" -o -iname "*self_rag*"
```
**⚠️ STALE/RESTRUCTURED.** All four curated links —
`docs/docs/tutorials/rag/langgraph_agentic_rag.ipynb`, `langgraph_adaptive_rag.ipynb`, `langgraph_crag.ipynb`, `langgraph_self_rag.ipynb` (and their `_local` variants) — 404 on `main` as of 2026-07-13, part of the same LangGraph docs reorganization flagged throughout this portfolio. **This is the single most load-bearing set of references in this entire project** — it's worth real effort to `find` their current location (or browse LangGraph's live docs site with normal network access at implementation time) rather than reimplementing all 4 patterns purely from the conceptual descriptions in PROFESSOR-NOTES.md. If you truly cannot locate the updated notebooks, the conceptual descriptions in this project's PLAN.md/PROFESSOR-NOTES.md are sufficient to implement each pattern correctly — treat the tutorials as a time-saver, not a hard dependency.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| RAGAS | Reuse as-is — your scoring engine for the whole comparison | Phase 5 |
| LangGraph Agentic RAG tutorial (once located) | Adapt — your Phase 4 implementation | Phase 4 |
| LangGraph Adaptive RAG tutorial (once located) | Adapt — your Phase 3 implementation | Phase 3 |
| LangGraph Corrective RAG (CRAG) tutorial (once located) | Adapt — your Phase 1 implementation; also the basis of Project 01's Fundamentals agent if built first | Phase 1 |
| LangGraph Self-RAG tutorial (once located) | Adapt — your Phase 2 implementation | Phase 2 |

## 3. Corpus reuse

If Project 01 is already built, reuse its ingested SEC filings as this project's shared corpus (PLAN.md §0/§5) — this keeps setup cost low and lets you compare RAG architectures on a corpus you already understand well. Otherwise, any moderate (10-30 document) technical corpus works; just ensure it's rich enough to contain genuine multi-hop questions (information split across 2+ documents) for the "multi_hop" question category.
