# RESOURCES.md — Codebase Onboarding Agent

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### tree-sitter + Python bindings
```bash
git clone --depth 1 https://github.com/tree-sitter/tree-sitter.git    /tmp/ref-tree-sitter      # ✅ master
git clone --depth 1 https://github.com/tree-sitter/py-tree-sitter.git /tmp/ref-py-tree-sitter   # ✅ master
```
Both default to **`master`** (not `main`). Your multi-language AST-aware chunking library (PLAN §0/§8). If you scope to Python only, use the stdlib `ast` module and skip this — document that scoping in your README.

### AutoGen RetrieveChat (RAG over a codebase, read-only, legacy)
```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_RetrieveChat.ipynb   # ✅ 200
```
⚠️ `0.2` = legacy. Read for the retrieval-augmented Q&A flow; you differ by using AST-aware chunks + citation verification + hybrid search instead of its default chunking.

### LangGraph code-assistant tutorial (read-only)
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Moved to `examples/` — verified: **`examples/code_assistant/langgraph_code_assistant.ipynb`** ✅ (and `..._mistral.ipynb`). It's about generating/fixing code, not Q&A over an existing codebase — useful mainly for the self-correction control flow if you extend the Issue-Finder to also propose fixes.

### Agno README-generator reference (read-only)
```bash
git clone --depth 1 https://github.com/agno-agi/agno.git /tmp/ref-agno
```
⚠️ The old `cookbook/examples/agents/readme_generator.py` is **deleted** (cookbook renumbered). For a comparable "summarize into a document" pattern, read agent examples under `cookbook/02_agents/`. Pattern reference only.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| tree-sitter + py-tree-sitter | both repos (master) | Reuse — chunking library | 0, 1 |
| AutoGen RetrieveChat (0.2) | `notebook/agentchat_RetrieveChat.ipynb` | Read-only — retrieval flow | 2 |
| LangGraph code-assistant | `examples/code_assistant/langgraph_code_assistant.ipynb` | Read-only — self-correction flow (stretch) | 4 |
| Agno agents cookbook | `cookbook/02_agents/` | Read-only — summarize-to-doc prompt | 3 |

## 3. Test-repo selection (data)
Pick 5 varying in size/language: one small (<50-file) Python lib, one ~500-file web app, one second-language repo (if multi-lang tree-sitter), one with an already-good README (sanity-check you don't make it worse), and one you've **never** looked at (held out for the live-demo robustness test in the DoD).

## 4. Also useful
- `rank-bm25` (`pip install rank-bm25`) — the sparse half of the hybrid retriever.
- `shrey1409/500-AI-Agents-Projects` → `agents/02-code-review-agent/` — a single-file code agent; skim for prompt patterns.
