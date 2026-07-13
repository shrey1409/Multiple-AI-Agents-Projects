# PROFESSOR-NOTES.md — Codebase Onboarding Agent

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Abstract syntax trees (ASTs), at a conceptual level | Your chunker's whole value proposition is parsing by AST boundary, not text | Python's own `ast` module docs, or `tree-sitter`'s "Playground" to see parse trees visually |
| RAG fundamentals (chunking, embedding, retrieval) | Same underlying mechanics as Project 01, applied to code instead of prose | Reuse what you learned in Project 01/prerequisites there |
| Static analysis basics (what an import graph is) | The architecture-diagram generator is built on this | Any short primer on Python/JS module dependency graphs |

## 2. Core Concepts Taught

### AST-aware chunking
**What:** splitting source code into retrieval units aligned with its actual syntactic structure (a whole function, a whole class) rather than splitting every N characters/tokens regardless of what's in the middle.
**Why it exists:** fixed-size splitting on code frequently cuts a function in half, separating a function's signature from its body, or splitting a class from its methods — this destroys both retrieval quality (the chunk doesn't make sense alone) and citation accuracy (you can't honestly cite "lines 40-60" if the function actually spans 35-72).
**How it works:** parse the file into an AST (or use `tree-sitter`'s language grammars for multi-language support), walk the tree to find function/class/module-level nodes, and emit one chunk per node with its exact start/end line numbers as metadata.
**Where it's used here:** the AST-Aware Chunker — the component that makes the whole project's citation-accuracy claim possible.

### Grounded citation (verify, don't trust)
**What:** treating an LLM's claimed citation (e.g., "see `utils.py:42`") as unverified until checked against the actual indexed content, and rejecting/regenerating answers that fail verification.
**Why it exists:** LLMs can produce syntactically plausible but factually wrong citations (a real-looking file path and line number that doesn't actually say what's claimed) — this is a subtler and more dangerous form of hallucination than an obviously wrong answer, because it looks trustworthy.
**Where it's used here:** the hard code-level check in the Q&A Agent's output path (PLAN.md §8) — this is a deterministic software check, not something you ask the LLM to self-verify.

### Static analysis as a complement to LLM reasoning
**What:** using exact, deterministic program analysis (import graphs, TODO scanning) for facts that are exact, and reserving the LLM for synthesis/labeling/summarization on top of those facts.
**Why it exists:** an LLM asked to "figure out the architecture" from raw text will sometimes hallucinate a relationship that doesn't exist in the actual code; a static import graph cannot be wrong about which modules import which — combining the two gives you accuracy (from static analysis) plus readability (from the LLM's labeling/diagramming).
**Where it's used here:** the Architecture-Diagram Generator's two-stage design (static import graph, then LLM to label/simplify it into a mermaid diagram) and the Issue-Finder's TODO/gap heuristics.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Working with AST/tree-sitter parsers | A genuinely differentiated skill versus "I chunked text every 500 tokens" |
| 1 (Indexing) | Building a code-search index with rich metadata | Directly applicable to any "chat with your codebase" internal tool — a common enterprise ask |
| 2 (Q&A) | Citation verification as a deterministic check, not an LLM courtesy | Teaches the general principle of "verify what you can verify in code, judge only what you can't" |
| 3 (README+Diagram) | Combining static analysis with LLM synthesis | A pattern that generalizes far beyond this project — anywhere you have exact facts plus a need for a readable summary |
| 4 (Issue-Finder) | Turning static heuristics into actionable, scoped suggestions | Product sense: the difference between "technically correct" output and output someone can actually use |
| 5 (Eval) | Manual verification as a legitimate eval method at small scale | Not everything needs an LLM judge — sometimes checking 5 repos by hand is the right, honest choice |
| 6 (Deploy) | Handling an unpredictable, live, user-supplied input (an arbitrary repo) robustly | The most realistic "will this survive contact with a real user" test in the whole portfolio |

## 4. Common Misconceptions & Mistakes

- **Falling back to fixed-size chunking "just to get something working."** This quietly abandons the project's entire differentiator; if you do this temporarily, treat it as a known gap, not a finished Phase 0.
- **Trusting LLM-stated citations without verification.** The most common way this project fails a rigorous eval — an unverified citation that looks right but points to the wrong lines.
- **Asking the LLM to infer the whole architecture from text alone.** Wastes the fact that you have exact import-graph data available; use static analysis for what's exact.
- **Ignoring vendored/generated directories.** Indexing `node_modules` or `.venv` bloats your index, slows indexing past the 5-minute target, and produces noisy, low-value chunks.
- **Testing only on repos you've already seen.** The real test of "run this on the interviewer's repo" is running it on a repo you've never indexed before — do this before you consider the project done, not for the first time during an actual interview.

## Understanding-check questions

**After §2 (AST-aware chunking):** What specifically goes wrong — for both retrieval quality and citation accuracy — if you split a 40-line function into two 20-line chunks at an arbitrary character boundary?

**After §2 (Grounded citation):** Why is verifying a citation in code more reliable than asking the LLM "are you sure about that citation?" What's the difference in what's actually being checked?

**After §2 (Static analysis + LLM):** Give an example fact about a codebase's architecture that a static import graph can tell you with certainty, and one that it fundamentally cannot (and needs an LLM or a human to reason about instead).

**After Phase 4 (Issue-Finder):** What makes a suggested "good first issue" actionable versus vague? Rewrite a vague suggestion ("improve error handling") into a scoped one using this project's approach.

**After Phase 6 (Deploy):** You run the agent live on a repo you've never seen and it times out during indexing. What are the first two things you'd check, based on the risks listed in this plan?
