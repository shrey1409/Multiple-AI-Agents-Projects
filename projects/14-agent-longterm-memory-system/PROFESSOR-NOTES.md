# PROFESSOR-NOTES.md — Agent Long-Term Memory System

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Embeddings + vector search (cosine) | Retrieval and dedup rest on it | Reuse Project 01's embedding prerequisite |
| pgvector basics | Your store is Postgres + vectors | pgvector README |
| Project 04's memory-lite | This is its deep version | Your own Project 04 |
| The human memory taxonomy (episodic/semantic/procedural), conceptually | You model these three types | Any short cognitive-science summary — you're borrowing the vocabulary, not the biology |

## 2. Core Concepts Taught

### Why "embed everything" is not memory
**What.** The naive approach: embed every message, retrieve top-k by similarity. It's a *retrieval* system, not a *memory* system.
**Why it exists as a foil.** It has no notion of what's worth keeping, no distillation of repeated signals into a stable fact, no handling of "the user changed their mind," and it grows unbounded. It's the baseline you must beat, and beating it is what proves you built memory.
**Where here.** The naive baseline in the benchmark.

### The three memory types
**What.** **Episodic** = specific time-stamped events; **semantic** = distilled facts/preferences; **procedural** = learned how-tos.
**Why it exists.** They're retrieved differently and have different lifetimes: episodic memories are numerous and decay; semantic memories are the durable distillation you actually want as background context; procedural memories shape behavior. Collapsing them into one flat store (Project 04's approach) loses these distinctions.
**Where here.** The typed `Memory` schema and type-aware retrieval.

### Consolidation (how an agent "learns over time")
**What.** Periodically distilling clusters of episodic memories into a semantic memory.
**Why it exists.** "Learning over time" is vague until you make it mechanical: three episodes of the user choosing email → one semantic "prefers email." Without consolidation, the agent re-derives the preference from raw episodes every time (expensive, unreliable) or misses it.
**How it works (mechanism).** A batch job clusters related episodic memories (by embedding proximity + recency), summarizes each cluster into a semantic memory, and records `source_episode_ids` for provenance. Retrieval then prefers the compact semantic memory.
**Where here.** The Consolidation job.

### Forgetting and supersession
**What.** **Decay:** salience drops with age unless reinforced by access. **Supersession:** a new fact that contradicts an old one marks the old `superseded_by`, and retrieval excludes it.
**Why it exists.** Unbounded memory degrades retrieval (more noise) and cost; and returning a stale, contradicted preference as current is the most visible memory failure. Forgetting keeps memory relevant; supersession keeps it correct. Note: supersession *marks*, doesn't delete — you keep the history for audit ("the user used to prefer phone, changed to email on X").
**Where here.** The decay job + contradiction handling.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Typed memory schema in pgvector | The data model behind stateful agents |
| 1 | Write policy: extraction, dedup, classification | What to store — the decision "embed everything" skips |
| 2 | Type-aware bounded retrieval | Assembling context under a token budget |
| 3 | Consolidation into semantic memory | The mechanics of "learning over time" |
| 4 | Decay + supersession | Keeping memory relevant and correct |
| 5 | Benchmarking memory vs. a baseline | Proving memory *helps*, which most demos never do |
| 6 | Shipping memory as a reusable service | A plug-in other agents adopt |

## 4. Common Misconceptions & Mistakes

- **"Memory = a bigger vector store."** That's retrieval; memory needs write/consolidate/forget policies.
- **No consolidation.** The agent accumulates but never learns.
- **Deleting instead of superseding.** Loses auditable history.
- **Unbounded retrieval.** Blows the context budget; rank by salience and bound it.
- **No eval.** Without beating a baseline, "it has memory" is unfalsifiable.

## 5. Understanding-check questions (with answer key)

**Q1 (Baseline).** Why isn't "embed every message and retrieve top-k" a memory system? Name two things it can't do.
**A1.** It's pure retrieval — it (1) can't distill repeated signals into a stable fact (no consolidation, so "prefers email" is never formed, only scattered episodes), and (2) can't handle contradiction (a changed preference sits alongside the old one, and similarity search may return the stale one as current). It also grows unbounded.

**Q2 (Types).** Give a query best served by episodic memory and one best served by semantic. Why store them separately?
**A2.** Episodic: "what did I ask you last Tuesday about order #123?" — needs the specific time-stamped event. Semantic: "how do I like my summaries?" — needs the distilled preference, not every instance. Separately because they're retrieved on different triggers and have different lifetimes (episodic decays, semantic persists).

**Q3 (Consolidation).** Describe mechanically how three episodes become one semantic memory. Why cite source episodes?
**A3.** A batch job clusters the three related episodic memories (embedding proximity), summarizes them into "prefers email," writes that as a semantic memory, and records the three `source_episode_ids`. Citing sources gives provenance — you can explain *why* the agent believes the preference and audit/revise it if the episodes were misread.

**Q4 (Supersession).** Why mark `superseded_by` instead of deleting the old memory? What breaks if retrieval ignores supersession?
**A4.** Marking preserves history (the user *used to* prefer phone) for audit and for understanding change over time, while excluding it from current retrieval. If retrieval ignores supersession, the agent can return the contradicted, stale preference as current — the most visible and trust-destroying memory failure.

**Q5 (Eval).** Why benchmark against a naive baseline instead of just showing the agent "remembers things"?
**A5.** "It remembered X" is a cherry-picked anecdote. A benchmark on a multi-session set — especially the consolidation/supersession subset where naive embedding fails — quantifies the *marginal value* of your policies (e.g., +15pp), turning a demo into evidence.
