# PROFESSOR-NOTES.md — Multi-Agent Financial Research Analyst (India / NSE-BSE)

Taught as if you're a student who knows single-agent LLM apps but hasn't built a real multi-agent graph yet. Where a concept has a mechanism underneath (not just a name), we teach the mechanism.

> **India-first note.** Every *concept* below is unchanged from the original curriculum — supervisor orchestration, CRAG, reflection, structured output. The localization adds one new engineering concept (**ports & adapters**, §2) and swaps the domain examples to Indian markets. Nothing was removed. Where the original taught 10-K structure, you now learn Indian annual-report structure — same skill (section-aware retrieval), different document.

## 1. Prerequisites

| Concept | Why you need it first | Specific resource |
|---|---|---|
| LangChain tool calling + structured output (Pydantic) | Every agent is a tool-using LLM call; the router/critic are typed outputs | LangChain docs → "How to return structured data from a model" and "How to use tools" (current docs.langchain.com) |
| LangGraph `StateGraph` (nodes, edges, reducers, `Command`) | The whole project is one `StateGraph` with a cycle | LangGraph docs → "Quickstart" and "Low-level conceptual guide → Graphs"; read before any notebook |
| Embeddings + cosine similarity (the actual formula) | You can't reason about CRAG's grade step without knowing what retrieval returns and why | Any dense-retrieval primer; know that similarity = `A·B / (‖A‖‖B‖)` and that it's dot product of L2-normalized vectors |
| Indian annual-report structure (MD&A, Directors' Report, BRSR, Notes, Corporate Governance) + SEBI LODR quarterly-results format | Your chunking is section-aware; you must know what the sections mean to split on them | Read one real annual report (e.g. TCS or Infosys FY2024, free on the investor-relations page); SEBI LODR Regulations 30/31/33 overview for what announcements/shareholding/results filings contain |
| Basic India market literacy — promoter holding & pledging, FII/DII, circuit/price bands, lakh/crore, SEBI as regulator | The Report Generator and Critic reason with these as domain facts | Zerodha Varsity (free) modules on fundamental analysis + Indian market structure |

## 2. Core Concepts Taught

### Supervisor pattern
**What.** One central LLM-driven router decides which specialist runs next; specialists report back only through shared state.
**Why it exists.** With N free-talking agents you get up to N² communication paths and no single place to enforce "don't run a worker twice" or "stop after 4." A supervisor centralizes control flow.
**How it works (mechanism).** The router is not a bigger prompt — it's a **structured-output call**. Tool-calling APIs enforce a JSON schema by constraining the decoder: the model can only emit tokens consistent with the schema (constrained decoding / grammar-masked sampling), so `next` is guaranteed to be one of your `Literal` values. Contrast a ReAct loop, where the next action is parsed from free text: across T turns with per-turn parse-failure probability p, reliability is ~(1−p)^T — error compounds. A single-shot typed router removes the loop and the parsing.
**Where here.** The Supervisor node gating all four workers + the Report Generator.

### Corrective RAG (CRAG)
**What.** RAG that grades its retrieved chunks before generating, and falls back to web search when the grade is poor.
**Why it exists.** Naive RAG trusts whatever the retriever returns; if nothing relevant is in the store, the LLM confidently hallucinates from irrelevant chunks.
**How it works.** retrieve top-k → a separate grader call scores each chunk (here: **binary** 0/1, decided in PLAN §8) → if mean grade < 0.5, call the web-search tool as a fallback source. The grader failing is itself a failure mode: a wrongly-lenient grader keeps bad chunks (silent hallucination), a wrongly-strict grader triggers web fallback constantly (fundamentals barely use the filing). This is why we track `retrieval_grade` and `used_web_fallback` in state — so you can *see* which failure you have.
**Where here.** The Fundamentals agent — the only worker required to use CRAG.

### Reflection (critic loop)
**What.** A second LLM pass critiques the first against an explicit rubric and can send it back.
**Why it exists.** First-draft output is subtly wrong (missing hedges, unsupported claims, inconsistent numbers) in ways a differently-prompted second pass catches better than trying to be perfect in one shot.
**How it works.** Report Generator → `draft_report`; Critic returns `CritiqueVerdict{approved, score, dimension_scores, feedback}`; if not approved and under the cap, loop back with the feedback injected. Bounded to 2 revisions because unbounded reflection is a classic non-terminating failure.
**Where here.** The Critic node.

### Structured output / constrained decoding
**What.** Forcing an LLM response into a typed schema instead of free text.
**Why it exists.** Free-text parsing ("next: market_data") is brittle — one stray word breaks routing.
**How (deeper).** The provider validates each sampled token against the schema and zeroes the probability of schema-violating tokens before sampling. That's why a typed router "can't" return an invalid worker name — it's not politeness, it's the decoder.
**Where here.** The router's `RouteDecision` and the critic's `CritiqueVerdict`.

### Ports & adapters (the localization's engineering lesson — new)
**What.** Isolating "what the system does" (the agents) from "where the data comes from" (the market) behind an interface, so the source is swappable.
**Why it exists.** Your agents shouldn't know or care whether prices come from NSE via yfinance `.NS` or from the SEC via EDGAR. If market-specific code leaks into the agents, supporting a second market means rewriting the agents — brittle in exactly the way real systems can't afford. The ports-and-adapters (a.k.a. hexagonal) pattern makes the *market* a plug-in.
**How it works (mechanism).** Define a **port** — a `Protocol`/interface like `MarketDataAdapter.get_market_data(ticker) -> MarketDataResult`. Write one **adapter** per market implementing it (India: yfinance `.NS` + nsepython; US: yfinance + SEC). Config selects the adapter at startup. The agents depend only on the port's type signature, never on a concrete source. Adding a market = adding an adapter, changing one config value — zero agent edits.
**Where here.** The `MarketDataAdapter` / `NewsAdapter` / `FilingsAdapter` layer (PLAN §2). This is why "worldwide mode" is a config flag, not a rewrite — and it's a genuinely hireable pattern (every fintech that serves multiple exchanges builds exactly this). Trade-off: one extra indirection layer for total source-swappability; worth it the moment you have >1 data source.
**Where it goes next.** The same pattern reappears whenever a system must support multiple providers (payment gateways: Razorpay vs Stripe; SMS: MSG91 vs Twilio; brokers: Kite vs Upstox). Naming it now means you recognize it everywhere later.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Section-aware chunking (Indian annual-report structure), vector ingestion, ports-&-adapters data layer | RAG ingestion is on nearly every AI-eng JD; the adapter pattern is what fintechs building multi-exchange systems test for |
| 1 | Minimal LangGraph state machine | The pattern every other portfolio project reuses |
| 2 | Tool design + isolating/unit-testing one agent | "How do you test an agent?" — most candidates have no answer |
| 3 | Multi-agent routing + shared-state design | The #1 cited agent-engineering skill gap |
| 4 | Self-critique loops, golden sets, rubric design | Separates you from "I called an LLM in a loop" |
| 5 | Containerizing + serving a multi-agent system behind an API | Recruiters filter on "did you deploy this" |
| 6 | Writing a technical narrative recruiters read | The README is often the only artifact examined >30s |

## 4. Common Misconceptions & Mistakes

- **"More agents = better."** A 4-worker supervisor isn't smarter than one good agent with 4 tools; the value is failure isolation + independent testing, not agent count.
- **Critic as rubber stamp.** A vague rubric ("is this good?") produces noise. Rubric items must be checkable facts (all 4 sections present, citations resolve, numbers match `quant_analysis`).
- **"CRAG fixes bad retrieval."** It only tells you *when* retrieval was bad and gives a fallback; bad chunking → constant web fallback → fundamentals barely use the filing.
- **Supervisor that also synthesizes.** Routing and writing are different jobs; merging them balloons the router schema and makes it unreliable.
- **No turn cap.** Works in the demo, hangs on an edge-case ticker.
- **Market logic leaking into agents (India context).** Hardcoding `.NS` or an NSE quirk inside the Market-Data agent instead of behind the adapter — it feels faster now and makes "worldwide mode" a rewrite later. Keep market specifics in the adapter.
- **lakh/crore mis-scaling.** Treating a "₹2,340 cr" revenue as 2,340 (not 2.34e10) corrupts every downstream ratio. Store raw floats; format only at the view layer.
- **Trusting an undisclosed pledge as zero.** If the shareholding filing doesn't disclose promoter pledge, the agent must say "not disclosed," not "0%." Absence of data ≠ a value.

## 5. Understanding-check questions (with answer key)

**Q1 (Supervisor).** Why does central routing scale better than letting workers pick who's next, and what breaks if two workers can call each other?
**A1.** Central routing keeps control-flow logic in one place (one turn cap, one "don't-repeat" rule) and turns N² potential paths into N. If workers call each other, no single node can enforce termination or dedup, and you can get mutual-call cycles with no cap.

**Q2 (CRAG).** If the grader is wrong about a chunk, what's the failure — better or worse than not grading?
**A2.** A lenient grader keeps irrelevant chunks → silent hallucination (worse than obvious "no answer"). A strict grader over-triggers web fallback → filing under-used. Not grading is uniformly "trust whatever came back"; grading can be *worse* when miscalibrated, which is why you measure `retrieval_grade`.

**Q3 (Reflection).** Why cap the critic instead of looping until approved? What do you do with a never-approved report?
**A3.** LLM critics can be unsatisfiable (each pass finds a new nit) → non-termination + cost blowup. Cap at 2; ship the best draft with a visible "unresolved critique" note so the failure is honest, not hidden.

**Q4 (Eval).** Why is citation integrity code-checked while grounding is LLM-judged?
**A4.** Citation integrity is a deterministic fact (does index `n` exist in `sources`) — code is exact and free. Grounding ("is the claim supported") is open-ended judgment with no exact-match answer, so it needs a model. Principle: verify deterministically what you can; judge only what you can't.

**Q5 (Deploy).** A recruiter has 90 seconds. What three things do they check first, and does your README answer them?
**A5.** (1) What does it do + a diagram; (2) does it work — the eval numbers table; (3) did you deploy it — a live link. If any is missing, the README fails the 90-second test.

**Q6 (Ports & adapters).** Your Market-Data agent has `if ticker.endswith(".NS"): ...` inside it. Why is that a design smell, and where should that logic live?
**A6.** The agent now knows market-specific detail, so a second market means editing the agent — the coupling the adapter pattern exists to prevent. The `.NS` handling belongs in the India `MarketDataAdapter`; the agent should call `get_market_data(ticker)` and receive a `MarketDataResult` regardless of market. Then "worldwide mode" is selecting a different adapter, not touching the agent.

**Q7 (India domain).** The Consistency rubric dimension fails a report that says "revenue ₹2,340 lakh" when the filing says "₹2,340 crore." Is the Critic right, and what's the underlying bug?
**A7.** The Critic is right — a lakh/crore mismatch is a 100× error in a stated figure, exactly the kind of internal inconsistency the dimension catches. The underlying bug is almost always unit handling: either the ingestion read the wrong scale from the statement, or the report layer formatted a raw float with the wrong Indian-numbering suffix. Fix at the boundary (parse/format), not by prompting the model to "be careful."
