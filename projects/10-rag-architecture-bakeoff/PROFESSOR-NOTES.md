# PROFESSOR-NOTES.md — RAG Architecture Bake-Off

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Naive RAG (retrieve top-k, stuff prompt) | All variants refine this baseline | Any intro RAG tutorial |
| RAGAS metric definitions | You report these numbers | RAGAS docs → "Metrics"; know the formulas below |
| Experiment design (controls, confounds) | The project's validity depends on it | Any A/B-testing / controlled-comparison primer |
| Basic significance (CI on a small sample) | ~13 Qs/category is small; don't over-claim | Any intro to confidence intervals |

## 2. Core Concepts Taught

### The four variants (and what each *mechanism* buys over naive)

**Agentic RAG.** An agent decides, per query, *whether* to retrieve and *what* strategy (skip, single, iterate). Buys: no wasted retrieval when the model already knows; escalation for multi-hop. Watch: it must actually vary behavior by query, or it's not agentic.

**Adaptive RAG.** Routes by an *upfront query-complexity classification* (no-retrieval / single / multi-step). Buys: spend retrieval budget where needed, save latency on easy Qs. Distinguished from Agentic by *when* the decision happens (once, upfront, on complexity) vs. Agentic's mid-loop, strategy-level choice.

**Corrective RAG (CRAG).** Grades retrieved chunks *before* generating; web-fallback on poor grade. Buys: catches "nothing relevant in the corpus" before the model hallucinates from junk.

**Self-RAG.** Reflects on the *generated answer's* groundedness; re-retrieves if unsupported. Buys: catches a different failure than CRAG — good retrieval but the model drifted from context while writing.

CRAG checks *retrieval* pre-generation; Self-RAG checks the *answer* post-generation. A question with good chunks but a drifting answer is caught by Self-RAG, not CRAG; a question with no relevant chunks is caught by CRAG's grade, not Self-RAG's answer-check (which might pass a confidently-wrong answer).

### RAGAS metrics (what they actually compute)
- **Faithfulness:** fraction of claims in the answer that are supported by the retrieved context (answer-vs-context grounding). Low = hallucination.
- **Answer relevancy:** how well the answer addresses the question (answer-vs-question).
- **Context precision:** of the retrieved chunks, how many are actually relevant (retrieval quality, precision).
- **Context recall:** of the info needed (per the gold answer), how much the retrieved context covers (retrieval quality, recall).
Faithfulness/relevancy grade *generation*; precision/recall grade *retrieval* — which is why you report all four: a variant can retrieve well but generate unfaithfully, or vice versa.

### Controlled experimentation
**What.** Hold everything constant except the one variable under test (here: RAG control-flow), so observed differences are attributable to it.
**Why it exists.** Without it, a comparison is confounded — you can't tell if CRAG "won" because of its grader or because it happened to chunk differently.
**Where here.** The frozen Phase-0 foundation — the single most important methodological decision.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Controlled-experiment design for AI | Rare rigor signal — most RAG projects report one number with no baseline |
| 1 | Retrieval grading + fallback + a naive baseline | Deepens Project 01's CRAG, studied in isolation |
| 2 | Post-generation groundedness checking | A distinct self-correction pattern |
| 3 | Query-complexity routing | Direct cost/latency optimization for any RAG deployment |
| 4 | Agent-driven retrieval strategy | The most "agentic" variant |
| 5 | Running + interpreting a multi-system eval | "I can tell you which RAG to use and why, with numbers" |
| 6 | Communicating a category-dependent result | Avoids "my system got 0.85" with no context |

## 4. Common Misconceptions & Mistakes

- **Agentic ≈ Adaptive.** Distinct (see §2); collapsing them is the #1 mistake.
- **Varying corpus/chunking/embeddings per variant.** Invalidates the comparison.
- **Aggregate-only scores.** Hide which variant wins where.
- **RAGAS metrics as ground truth.** Model-graded; use a different judge model.
- **Skipping no-answer questions.** Misses the hallucinate-when-nothing-exists failure.

## 5. Understanding-check questions (with answer key)

**Q1 (Agentic vs. Adaptive).** Structural difference? Where in each impl does it show?
**A1.** Adaptive: a classifier node runs *once at the start* and routes to a fixed path by complexity. Agentic: an agent loop decides retrieval per step and can re-retrieve mid-process. In code, Adaptive has an upfront router→branch; Agentic has a loop with a retrieve-or-answer decision each iteration.

**Q2 (CRAG).** What failure does CRAG's grading catch that naive misses? Concrete example.
**A2.** Retrieval returning nothing relevant. Example: ask about a company not in the corpus — naive stuffs irrelevant chunks and the model hallucinates; CRAG grades them low (mean<0.5) and web-falls-back or abstains.

**Q3 (Self-RAG vs. CRAG).** How is Self-RAG's re-retrieval trigger different from CRAG's fallback trigger? A case exposing the difference?
**A3.** CRAG triggers on poor *retrieval grade* (pre-generation); Self-RAG triggers on the *answer* being unsupported (post-generation). Case: good chunks retrieved (CRAG passes) but the model writes a claim not in them — Self-RAG's answer-check catches the drift; CRAG never looks at the answer.

**Q4 (Controls).** Name three things held constant across variants; what conclusion is impossible if any varies?
**A4.** Embedding model, chunking, corpus/index. If chunking varied, you couldn't conclude a variant's win came from its control-flow rather than a lucky chunk size — the comparison is confounded.

**Q5 (Reporting).** Aggregate faithfulness is nearly identical across variants but the category breakdown differs a lot. Which do you report, and why?
**A5.** The category breakdown. The aggregate averages away the actual signal (e.g., CRAG wins on no-answer, ties elsewhere). The useful, honest finding is per-category; the aggregate would make the report look boring and hide the recommendation.
