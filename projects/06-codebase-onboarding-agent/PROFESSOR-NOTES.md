# PROFESSOR-NOTES.md — Codebase Onboarding Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Abstract syntax trees (conceptually) | Your chunker parses by AST boundary | Python `ast` module docs; the tree-sitter Playground to see parse trees |
| RAG fundamentals (chunk/embed/retrieve) | Same mechanics, applied to code | Reuse Project 01's RAG prerequisite |
| tree-sitter queries (S-expressions) | You walk the tree to find function/class nodes | tree-sitter docs → "Using Parsers" and "Query syntax" |
| Static analysis / import graphs | The diagram generator is built on this | Any primer on module dependency graphs |

## 2. Core Concepts Taught

### AST-aware chunking
**What.** Split code into retrieval units aligned with syntax (a whole function/class), not every N characters.
**Why it exists.** Fixed-size splitting cuts a function in half — separating signature from body — destroying both retrieval quality (the chunk is meaningless alone) and citation accuracy (you can't honestly cite lines 40–60 of a function spanning 35–72).
**How it works (mechanism).** Parse the file to an AST; with tree-sitter you run an **S-expression query** like `(function_definition) @fn` to capture function nodes across languages, then emit one chunk per node with its exact byte/line span. Oversized nodes (a huge class) get split by member, each keeping the parent symbol.
**Where here.** The AST-Aware Chunker — the component the whole citation claim rests on.

### Why code embeddings differ from prose embeddings
**What.** Code retrieval needs identifier-exact matching that dense vectors alone under-serve.
**Why it exists.** An identifier like `parse_config` is a rare token; a dense embedding may place two functions near each other semantically while the user actually wants the exact symbol. BM25 (term-frequency over tokens) nails exact-identifier lookups; dense nails conceptual ("how does auth work"). Fusing both (reciprocal-rank fusion) covers both query types.
**Where here.** The hybrid retriever.

### Grounded citation (verify, don't trust)
**What.** Treat an LLM's citation as unverified until checked against the actual indexed chunk.
**Why it exists.** LLMs produce plausible-but-wrong citations — a real-looking `utils.py:42` that doesn't say what's claimed. This is subtler than an obvious wrong answer because it looks trustworthy.
**How it works.** Deterministic check: parse the citation, assert the `file:line` maps to a real chunk in the index; regenerate if not. That's software, not asking the LLM "are you sure".
**Where here.** The hard assertion in the Q&A output path.

### Static analysis + LLM (exact facts + readable synthesis)
**What.** Use deterministic program analysis for exact facts (import graph, TODO scan), the LLM for labeling/summary on top.
**Why it exists.** An LLM asked to "figure out the architecture" hallucinates edges; a static import graph cannot be wrong about which module imports which. Combine: accuracy from analysis, readability from the LLM.
**Where here.** The two-stage Architecture-Diagram Generator and the Issue-Finder heuristics.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | AST/tree-sitter parsing | Differentiated vs. "I chunked every 500 tokens" |
| 1 | Hybrid code-search index with metadata | Any "chat with your codebase" internal tool |
| 2 | Citation verification as a deterministic check | "Verify what you can; judge only what you can't" |
| 3 | Static analysis + LLM synthesis | Generalizes anywhere exact-facts + readable-summary meet |
| 4 | Static heuristics → scoped suggestions | Product sense: usable vs. merely correct output |
| 5 | Manual verification as legitimate at small scale | Not everything needs an LLM judge |
| 6 | Robustness on live, unpredictable input | The most realistic "survives a real user" test |

## 4. Common Misconceptions & Mistakes

- **Fixed-size fallback "just to ship."** Abandons the differentiator; treat as a known gap.
- **Trusting LLM citations.** The most common rigorous-eval failure.
- **LLM-inferred architecture.** Wastes the exact import-graph data.
- **Ignoring vendored dirs.** Bloats the index and times out live demos.
- **Testing only seen repos.** Run on an unseen one before "done".

## 5. Understanding-check questions (with answer key)

**Q1 (AST chunking).** What goes wrong — retrieval and citation — if you split a 40-line function at an arbitrary character boundary?
**A1.** Retrieval: each half lacks context (signature without body, or body without signature), so neither chunk is a good standalone answer. Citation: you'd cite a line range that doesn't correspond to a complete semantic unit, so "see lines 40–60" misrepresents where the function actually is.

**Q2 (Grounded citation).** Why is code-verifying a citation more reliable than asking the LLM "are you sure"?
**A2.** Asking the LLM re-queries the same fallible model that produced the citation — it can be confidently wrong twice. Code checks an external fact (does this file:line exist in the index and map to a real chunk), which is deterministic and can't be talked into a wrong answer.

**Q3 (Static + LLM).** Give one architecture fact a static import graph tells you with certainty and one it can't.
**A3.** Certain: module A imports module B (it's in the source). Cannot: *why* the design is layered that way, or runtime relationships via dynamic dispatch/dependency injection/`importlib` — those need an LLM or a human to reason about.

**Q4 (Issue-Finder).** What makes a "good first issue" actionable vs. vague? Rewrite "improve error handling."
**A4.** Actionable = specific location + concrete change. Rewrite: "wrap the `open()` call in `loader.py:34` in a try/except and raise a `ConfigError` with the file path, since a missing config currently crashes with a bare `FileNotFoundError`."

**Q5 (Deploy).** It times out indexing an unseen repo. First two things to check?
**A5.** (1) Is it trying to index vendored/generated dirs (`node_modules`, `.venv`)? Add exclusions. (2) Is there an oversized file/monorepo blowing the file-count/size ceiling? Apply the size cap and skip binary/generated files.
