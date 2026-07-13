# PROFESSOR-NOTES.md — Agent Observability Stack

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Distributed tracing basics (span, trace, parent/child) | The whole project is tracing an agent | OpenTelemetry docs → "Observability primer" and "Traces" |
| OpenTelemetry SDK + OTLP + Collector | Your export path | OpenTelemetry Python docs → "Getting Started" + "Collector" |
| GenAI semantic conventions | The attribute schema you must follow | OpenTelemetry "Semantic Conventions → Generative AI" |
| Project 01's architecture | It's the agent you instrument | Your own Project 01 |

## 2. Core Concepts Taught

### Distributed tracing (spans and traces)
**What.** A **span** is a timed unit of work with attributes and a parent; a **trace** is the tree of spans for one request. For an agent, the root is a run, children are graph nodes, grandchildren are tool calls.
**Why it exists.** When an agent is slow, expensive, or wrong, you need to see *where* — which node burned the tokens, which tool timed out, where the trajectory branched. Logs alone don't show the causal tree; traces do.
**How it works (mechanism).** Each unit opens a span (recording start), sets attributes, and closes it (recording end); the SDK propagates the parent context so children attach to the right parent. Spans export over **OTLP** to a collector, which routes them to a backend (Jaeger/Tempo). Because the format is standard, any OTel-compatible viewer works.
**Where here.** The span model in PLAN §2.

### Semantic conventions (why a standard schema matters)
**What.** Agreed attribute names for a domain — `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.tool.name`, etc.
**Why it exists.** If everyone names token counts differently (`tokens_in` vs `input_tokens` vs `prompt_tokens`), no dashboard or backend can aggregate across systems. Conventions make traces portable and tooling reusable — the difference between a bespoke logger and an industry skill.
**Where here.** Following the GenAI conventions so your traces work in any OTel backend.

### Cost attribution
**What.** Assigning the dollar/token cost of a run down to individual spans (this node cost $0.04, that tool call added 1,200 tokens).
**Why it exists.** "The report cost $0.20" is unactionable; "the critic loop cost 60% of it" tells you where to optimize. Per-span cost turns a total into a diagnosis.
**Where here.** Cost attributes on spans + the reconciliation eval (sum-of-spans within 5% of billed).

### Observability vs. evaluation (this project vs. Project 03)
**What.** Observability = what the system *did* internally (traces, metrics). Evaluation = whether the output was *good* (judgments).
**Why it matters.** They answer different questions and use different tooling. A failing eval tells you *that* quality dropped; the trace tells you *why* (a tool started timing out, a retry loop kicked in). Production needs both.
**Where here.** 13 emits the trace; 03 reads the output — the Contract connects them.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | OTel SDK + collector setup | Standard observability plumbing, not agent-specific |
| 1 | Instrumenting graph nodes with spans | The core "trace my agent" skill |
| 2 | Tool-call spans + attribute hygiene (redaction) | Safe instrumentation of side-effecting calls |
| 3 | Defining an observability contract | The interface production tooling builds on |
| 4 | Building trace views + cost/latency dashboards | The dashboards ops teams live in |
| 5 | Proving reuse across a second agent | Same rigor signal as the portfolio's other reuse proofs |
| 6 | Packaging + a real debugging story | "I found and fixed X by reading the trace" is a strong interview anecdote |

## 4. Common Misconceptions & Mistakes

- **Bespoke schema instead of conventions.** Kills portability.
- **Raw args/PII on spans.** Redact; spans are logs too.
- **Ignoring overhead.** Sample + export async; measure it.
- **Two sources of truth.** Derive the Contract trajectory from the spans.
- **"This is just Project 03."** No — observability vs. evaluation.

## 5. Understanding-check questions (with answer key)

**Q1 (Tracing).** What does a span tree show you about a slow agent run that a flat log doesn't?
**A1.** The causal/timing structure: which node is the parent of which tool call, how long each took, and where time actually accumulated (e.g., a 6s tool call nested under the fundamentals node). A flat log shows events in order but not the parent/child timing tree, so you can't attribute the slowness to a specific nested operation.

**Q2 (Conventions).** Why follow GenAI semantic conventions instead of naming attributes yourself?
**A2.** Portability and tooling reuse — any OTel-compatible backend/dashboard understands `gen_ai.usage.input_tokens`, so your traces work in Jaeger, Tempo, or a vendor tool without custom adapters, and you can aggregate across services. Custom names lock you into bespoke tooling and aren't a transferable skill.

**Q3 (Cost attribution).** Why is per-span cost more useful than a run total? Give a decision it enables.
**A3.** It localizes spend. Example: seeing the critic-reflection loop consumes 60% of a report's cost lets you decide to cap revisions or use a cheaper critic model — a total ($0.20) gives you no such lever.

**Q4 (Observability vs. eval).** Your eval (Project 03) shows success rate dropped. How does a trace help, and what might it reveal?
**A4.** The eval says *that* quality dropped; the trace says *why*. It might reveal the web-search tool started returning errors (so fundamentals fell back constantly), or a model change increased token usage and truncated context — root causes invisible to output-only evaluation.

**Q5 (Safety).** Why must you redact tool args before putting them on spans?
**A5.** Spans are exported and stored like logs; putting raw args there can leak PII, secrets, or query contents into the trace backend — a new exposure surface. Redaction (types/hashes, not values) keeps traces useful without turning them into a leak, same discipline as Project 11's findings-not-values.
