# PROFESSOR-NOTES.md — RAG Architecture Bake-Off

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Naive RAG (retrieve top-k, stuff into prompt) | All 4 variants are refinements on this baseline; you need to know what they're improving on | Any introductory RAG tutorial/blog post |
| RAGAS's metric definitions (faithfulness, answer relevancy, context precision/recall) | You're reporting these numbers and need to know what each actually measures | RAGAS's own documentation on its core metrics |
| Basic experiment design (controlling variables, what a confound is) | The entire project's validity depends on holding corpus/embeddings/chunking constant across variants | Any short primer on controlled comparison / A-B testing methodology |

## 2. Core Concepts Taught

### Agentic RAG
**What:** RAG where an agent decides, per query, *whether* to retrieve at all and *what* retrieval strategy to use (single lookup, multiple lookups, different sources), rather than always retrieving the same fixed way.
**Why it exists:** naive RAG retrieves unconditionally even when the LLM already knows the answer or when a single lookup isn't enough for a multi-hop question — an agentic decision layer can skip unnecessary retrieval or escalate to a more thorough strategy as needed.
**Where it's used here:** the Agentic RAG variant — the strategy-choosing step is the whole point; make sure your implementation actually varies its retrieval behavior based on the query, or it's not meaningfully "agentic."

### Adaptive RAG
**What:** routing incoming queries to a no-retrieval, single-step-retrieval, or multi-step-retrieval path based on an *estimated query complexity* classification, made explicitly before any retrieval happens.
**Why it exists:** not all questions need retrieval (a definitional question the model already knows) and not all questions are answerable in one retrieval hop (a comparison across two documents) — classifying complexity upfront lets you spend retrieval budget where it's needed and save latency/cost where it's not.
**Where it's used here:** the Adaptive RAG variant — distinguished from Agentic RAG by routing on *complexity classification made upfront*, versus Agentic RAG's *strategy choice that can also happen mid-process*; this project's report should make this distinction concrete with actual examples, since it's easy to blur in practice (see the misconception below).

### Corrective RAG (CRAG)
**What:** grading retrieved chunks for relevance before generating an answer, and falling back to an alternative retrieval source (commonly web search) when the grade is poor.
**Why it exists:** naive RAG trusts whatever the retriever returns, even when nothing relevant exists in the corpus — CRAG adds a self-check specifically on retrieval quality, with an escape hatch.
**Where it's used here:** the Corrective RAG variant — you built a simpler version of this already in Project 01's Fundamentals agent; this project isolates and evaluates it on its own, side by side with the other 3.

### Self-RAG
**What:** the model reflects on its own *generated answer* (not just the retrieved chunks) and re-retrieves if the answer isn't well-supported by what was retrieved.
**Why it exists:** CRAG checks retrieval quality before generation; Self-RAG checks the generated answer's groundedness *after* generation — catching a different failure mode (good retrieval, but the model still drifted from the retrieved context while writing the answer).
**Where it's used here:** the Self-RAG variant — the reflection-then-conditional-re-retrieval loop is what distinguishes it from CRAG; your report should be able to name a case where this specifically catches something CRAG wouldn't (or vice versa).

### Controlled experimentation as an evaluation methodology
**What:** holding every variable constant across the systems you're comparing except the one thing you're actually testing (here: the RAG control-flow pattern), so that observed differences in the results can be attributed to that one variable.
**Why it exists:** without this discipline, a comparison is confounded and its conclusions are unreliable — you can't tell if Corrective RAG "won" because of its grading step or because it happened to use a slightly different chunking strategy.
**Where it's used here:** the entire Phase 0 "shared foundation" design — the single most important methodological decision in this project.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Shared foundation) | Controlled-experiment design for AI systems | A rare and valuable rigor signal — most portfolio RAG projects report one number with no comparison baseline |
| 1 (CRAG) | Retrieval-quality grading and fallback design | Deepens what you built in Project 01, now studied in isolation |
| 2 (Self-RAG) | Post-generation groundedness checking | A distinct self-correction pattern from Project 01's report-level critic |
| 3 (Adaptive RAG) | Query-complexity routing | Directly useful for cost/latency optimization in any real RAG deployment |
| 4 (Agentic RAG) | Agent-driven retrieval strategy selection | The most "agentic" of the four — reinforces agentic decision-making design from Project 01 |
| 5 (Bake-off+RAGAS) | Running and interpreting a real multi-system evaluation | The specific, rare skill of "I can tell you which RAG architecture to use and why, with numbers" |
| 6 (Report) | Communicating a nuanced, category-dependent result clearly | Avoids the common trap of "my system got 0.85 faithfulness" with no comparative context |

## 4. Common Misconceptions & Mistakes

- **Believing Agentic RAG and Adaptive RAG are basically the same thing.** They're closely related but conceptually distinct (see §2 above); collapsing them into identical implementations is the single most common mistake in this project — be deliberate about the distinction and demonstrate it with examples.
- **Varying the corpus, chunking, or embedding model between variants "for convenience."** Immediately invalidates the comparison; lock these down once and never touch them per-variant.
- **Reporting only aggregate scores.** Hides the actually useful finding (which variant wins on which question type); always break down by category.
- **Assuming RAGAS's LLM-based metrics are ground truth.** They're model-graded, same caveat as the LLM-as-judge concept in Project 03 — consider a different judge model than your generation model.
- **Skipping the "no answer in corpus" question category.** This category tests a distinct and important failure mode (hallucinating an answer when none exists) that easy/multi-hop questions don't exercise.

## Understanding-check questions

**After §2 (Agentic vs. Adaptive RAG):** In your own words, what's the structural difference between these two variants? Point to the specific place in each implementation where that difference shows up in code.

**After §2 (Corrective RAG):** What specific failure mode does CRAG's grading step catch that naive RAG would miss? Give a concrete example question from your test set.

**After §2 (Self-RAG):** How is Self-RAG's re-retrieval trigger different from CRAG's fallback trigger? Could a single question expose a case where one catches a problem and the other doesn't?

**After §2 (Controlled experimentation):** Name three things you held constant across all 4 variants in this project, and explain what conclusion you couldn't draw if you'd let any one of them vary.

**After Phase 5 (Bake-off+RAGAS):** Your aggregate faithfulness scores are nearly identical across all 4 variants, but the category breakdown shows large differences. Which number would you actually report, and why?
