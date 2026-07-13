# RESOURCES.md — RAG Architecture Bake-Off

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### RAGAS (scoring engine)
```bash
git clone --depth 1 https://github.com/explodinggradients/ragas.git /tmp/ref-ragas   # ✅
pip install ragas
```
Primary scorer for the whole bake-off. Read its docs on the current API for faithfulness / answer_relevancy / context_precision / context_recall — **metric import names have changed across versions**; pin one version and check names against the installed package.

### LangGraph's four RAG tutorials (Agentic, Adaptive, Corrective, Self-RAG)
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
**Not 404 — moved to `examples/rag/`.** Verified present:
- `examples/rag/langgraph_agentic_rag.ipynb` ✅
- `examples/rag/langgraph_adaptive_rag.ipynb` ✅ (and `langgraph_adaptive_rag_local.ipynb`, `..._cohere.ipynb`)
- `examples/rag/langgraph_crag.ipynb` ✅ (and `langgraph_crag_local.ipynb`)
- `examples/rag/langgraph_self_rag.ipynb` ✅ (and `langgraph_self_rag_local.ipynb`, `..._pinecone_movies.ipynb`)

This is the most load-bearing reference set in the project — and it's fully recovered. Adapt each to your frozen config. (The per-variant control-flow specs in PLAN §2 are also sufficient to implement each pattern if you prefer not to follow the notebooks.)

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| RAGAS | repo / `pip install ragas` | Reuse — scoring engine | 5 |
| LangGraph Agentic RAG | `examples/rag/langgraph_agentic_rag.ipynb` | Adapt — Phase 4 | 4 |
| LangGraph Adaptive RAG | `examples/rag/langgraph_adaptive_rag.ipynb` | Adapt — Phase 3 | 3 |
| LangGraph CRAG | `examples/rag/langgraph_crag.ipynb` | Adapt — Phase 1 (and basis of Project 01's Fundamentals agent) | 1 |
| LangGraph Self-RAG | `examples/rag/langgraph_self_rag.ipynb` | Adapt — Phase 2 | 2 |

## 3. Corpus reuse
If Project 01 is built, reuse its ingested SEC filings as the shared corpus (keeps setup cheap on a corpus you understand). Otherwise any moderate (15–25 document) technical corpus — just ensure it contains genuine multi-hop content (facts split across ≥2 documents) so the `multi_hop` category is real.
