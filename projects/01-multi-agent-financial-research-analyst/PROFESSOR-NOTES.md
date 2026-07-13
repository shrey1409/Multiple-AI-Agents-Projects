# PROFESSOR-NOTES.md — Multi-Agent Financial Research Analyst

Taught as if you're a student who knows single-agent LLM apps but hasn't built a real multi-agent graph yet.

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| Basic LangChain (LLM calls, tools, structured output with Pydantic) | Every agent in this project is a LangChain-style tool-using LLM call | LangChain "Tool calling" and "Structured output" how-to guides on the current LangChain docs site |
| LangGraph `StateGraph` fundamentals (nodes, edges, state reducers) | The whole project is one `StateGraph` | LangGraph's own "Quickstart" — read this before touching any tutorial notebook, since notebook APIs drift (see RESOURCES.md) |
| Embeddings & vector similarity search basics | You can't reason about CRAG's "grade the retrieval" step without knowing what a retrieval even returns | Any short primer on cosine similarity + dense retrieval; skip if you've already built a RAG app |
| What a 10-K/10-Q filing contains (Item 1, 1A, 7, 7A sections) | Your chunking strategy is section-aware; you need to know what the sections mean | SEC's own "How to Read a 10-K" investor guide |

## 2. Core Concepts Taught

### Supervisor pattern
**What:** one central LLM-driven router node decides which specialist agent runs next; specialists never talk to each other directly, only report back to the supervisor through shared state.
**Why it exists:** without a router, N agents would need N² communication paths and no single place enforces "don't call the same worker twice" or "stop after 4 workers are done." A supervisor gives you one place to put control-flow logic.
**How it works:** the supervisor is a structured-output LLM call — not a bigger prompt, a Pydantic-typed decision (`next: Literal["market_data","news","fundamentals","quant","report","FINISH"]`). LangGraph routes based on that field via `Command(goto=...)`.
**Where it's used here:** the single Supervisor node in §2 of PLAN.md, gating access to all four workers and the report generator.

### Corrective RAG (CRAG)
**What:** a RAG variant that grades its own retrieved chunks before generating an answer, and falls back to web search when the grade is poor.
**Why it exists:** naive RAG blindly trusts whatever the retriever returns; if the vector store has nothing relevant (e.g., a filing wasn't ingested, or the question is about something not in the 10-K), the LLM will confidently hallucinate an answer from irrelevant chunks.
**How it works:** retrieve top-k → a separate "grader" LLM call scores each chunk's relevance (binary or 1–5) → if the aggregate grade is below threshold, trigger a web-search tool call as a fallback source instead of (or in addition to) the weak chunks.
**Where it's used here:** the Fundamentals Agent — it's the one worker explicitly required to use CRAG rather than plain retrieval.

### Reflection (critic loop)
**What:** a second LLM pass that critiques the first pass's output against an explicit rubric and can send it back for revision.
**Why it exists:** first-draft LLM output is often subtly wrong (missing hedge language, unsupported claims, inconsistent numbers) in ways a second, differently-prompted pass catches more reliably than trying to get everything right in one shot.
**How it works:** Report Generator produces `draft_report` → Critic scores it against a fixed rubric and returns `{approved, feedback, score}` → if not approved and under the revision cap, loop back to Report Generator with the feedback injected into its prompt.
**Where it's used here:** the Critic/Reflection node, bounded to 2 revisions (§7 risk: unbounded reflection loops are a classic failure mode).

### Structured output / tool calling
**What:** forcing an LLM response into a typed schema (Pydantic/JSON schema) instead of free text.
**Why it exists:** free-text parsing ("next: market_data") is brittle — a single stray word breaks your router. Structured output makes routing and critique verdicts programmatically reliable.
**Where it's used here:** the supervisor's routing decision and the critic's verdict — both are Pydantic models, never parsed strings.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Setup) | Section-aware document chunking, vector store ingestion | Nearly every "AI engineer" JD lists RAG ingestion pipelines as a core skill |
| 1 (Skeleton) | Minimal viable LangGraph state machine | This is the pattern you'll reuse in every other project in this portfolio |
| 2 (Specialists) | Tool design, isolating and unit-testing a single agent | Interviewers probe "how do you test an agent" — most candidates have no answer; you will |
| 3 (Orchestration) | Multi-agent routing, shared state design | This *is* the #1 skill gap the market cites for agent engineering roles |
| 4 (Reflection + Eval) | Self-critique loops, building a golden eval set, LLM-as-judge rubric design | Distinguishes you from "I called an LLM in a loop" candidates |
| 5 (Deploy) | Containerizing a multi-agent system, serving it behind an API | Recruiters explicitly filter for "did you deploy this" |
| 6 (Polish) | Writing a technical narrative recruiters actually read | The README is often the only artifact anyone looks at for >30 seconds |

## 4. Common Misconceptions & Mistakes

- **"More agents = better."** A 4-worker supervisor isn't inherently smarter than one well-prompted agent with 4 tools — the value is in isolating failure domains (a bad news-search doesn't corrupt fundamentals) and enabling independent testing, not raw agent count.
- **Treating the critic as a rubber stamp.** If the critic's rubric is vague ("is this good?"), its score is noise. Rubric items must be checkable facts (are all 4 sections present, do citations resolve, do stated numbers match `quant_analysis`), not vibes.
- **Believing CRAG "fixes" bad retrieval.** CRAG only tells you *when* retrieval was bad and gives you a fallback; if your chunking strategy is bad in the first place, CRAG will just fall back to web search constantly and your fundamentals section will barely use the actual filing.
- **Confusing "supervisor" with "orchestrator that also does the work."** The supervisor should route, not synthesize — that's the Report Generator's job. Mixing the two makes the supervisor's structured-output schema balloon and become unreliable.
- **Forgetting a turn cap.** Students routinely ship a version that works in the demo but hangs forever on an edge-case ticker because nothing caps supervisor turns or critic revisions.

## Understanding-check questions

**After §2 (Supervisor pattern):** Why does routing through a central supervisor scale better than letting each worker decide who to call next? What breaks if two workers are allowed to call each other directly?

**After §2 (CRAG):** If the grader LLM itself is wrong about a chunk's relevance, what's the failure mode — and is it worse or better than not grading at all?

**After §2 (Reflection):** Why cap the critic loop at a fixed number of revisions instead of looping until "approved == True"? What would you do with a report that never gets approved?

**After Phase 4 (Eval):** Why is "citation integrity" checked in code rather than by the LLM judge, while "grounding" is judged by the LLM? What's the general principle for deciding which checks should be deterministic vs. model-graded?

**After Phase 6 (Deploy/Polish):** A recruiter has 90 seconds to look at your project. What three things in the README will they check first, and does your current draft answer all three?
