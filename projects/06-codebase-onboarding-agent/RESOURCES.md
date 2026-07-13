# RESOURCES.md — Codebase Onboarding Agent

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### tree-sitter and its Python bindings

```bash
git clone --depth 1 https://github.com/tree-sitter/tree-sitter.git /tmp/ref-tree-sitter
git clone --depth 1 https://github.com/tree-sitter/py-tree-sitter.git /tmp/ref-py-tree-sitter
```
Status: ✅ CONFIRMED WORKING on the **`master`** branch for both (not `main`). This is your primary parsing library for multi-language AST-aware chunking (PLAN.md §0/§8). If you scope down to Python-only, you can skip this and use the standard library `ast` module instead — document that scoping decision in your own README if you take that shortcut.

### AutoGen RetrieveChat (RAG over a codebase)

```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_RetrieveChat.ipynb
```
Status: ✅ CONFIRMED WORKING (this exact notebook family verified 200 elsewhere in this portfolio's research; re-verify at implementation time since it's an unauthenticated public URL that could change). Read for its retrieval-augmented Q&A flow design; your version differs by using AST-aware chunks with citation verification instead of RetrieveChat's default chunking.

### LangGraph Code Assistant tutorial

```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
find /tmp/ref-langgraph -iname "*code_assistant*" -o -iname "*code-assistant*"
```
**⚠️ STALE/RESTRUCTURED.** Curated link (`docs/docs/tutorials/code_assistant/langgraph_code_assistant.ipynb`) 404s on `main` as of 2026-07-13 — same LangGraph docs reorganization noted throughout this portfolio. Run the `find` above to locate its current home.

### Agno Readme Generator Agent

```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
find /tmp/ref-agno/cookbook -iname "*readme*"
```
**⚠️ STALE/RESTRUCTURED.** Curated link (`cookbook/examples/agents/readme_generator.py`) 404s — same Agno cookbook restructuring noted throughout this portfolio (now numbered `00_quickstart`...`99_docs`). Run the `find` above; likely now under `02_agents/`.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| tree-sitter + py-tree-sitter | Reuse as-is — your chunking library | Phase 0, 1 |
| AutoGen RetrieveChat | Read-only — retrieval flow design reference; you rewrite the chunking/citation-verification layer yourself | Phase 2 |
| LangGraph Code Assistant tutorial (once located) | Read-only — this tutorial is about generating/fixing code, not Q&A over an existing codebase; useful mainly for its self-correction control-flow pattern if you extend the Issue-Finder to also propose fixes | Phase 4 (stretch) |
| Agno Readme Generator (once located) | Adapt — reuse its prompt structure for turning code summaries into a README; you feed it your own retrieved chunks instead of its default input | Phase 3 |

## 3. Test-repo selection (data, not code)

Pick 5 public repos varying in size/language for your eval set (PLAN.md §6) — e.g. one small (<50 files) Python library, one medium (~500 files) web app, one repo in a second language if you built multi-language `tree-sitter` support, one repo with an already-good README (to sanity-check your generator doesn't make things worse), and one repo you've genuinely never looked at before (held out for the live-demo robustness test in the Definition of Done).
