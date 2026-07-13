# PROFESSOR-NOTES.md — Local / Offline Open-Weight Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| One built portfolio agent (05 or 01) | You port it, not rebuild | Your own build |
| How LLM inference works (tokens, context, KV cache) | You reason about local speed/memory | Any inference-basics primer |
| Quantization (fp16 → int8/int4) at a concept level | The core quality/size tradeoff | llama.cpp quantization docs; a GGUF/quant primer |
| Constrained decoding / grammars | The structured-output mitigation | llama.cpp GBNF docs; the `outlines` README |

## 2. Core Concepts Taught

### Why local open-weight agents are a different problem
**What.** Running the agent's model yourself (Ollama/llama.cpp/vLLM) instead of calling a frontier API.
**Why it exists.** Privacy (data never leaves your network — healthcare, finance, air-gapped), cost (at high volume, amortized hardware beats per-token pricing), latency/availability (no network dependency), and control (pinned weights). These are real deployment drivers, and the engineering is genuinely different from calling an API.
**Where here.** The whole project — the ported agent runs on a local model with the constraints that creates.

### Quantization (the quality/size/speed tradeoff)
**What.** Storing model weights at lower precision (fp16 → int8 → int4) to fit less memory and run faster, at some quality cost.
**Why it exists.** A full-precision model may not fit consumer/edge hardware or hit throughput needs; quantization makes deployment feasible. But it's a *tradeoff* — too aggressive and quality drops. The engineering skill is choosing the level with data, not vibes.
**How it works (mechanism).** Weights are grouped and rounded to a lower-bit representation (e.g., Q4_K_M) with per-group scales; inference dequantizes on the fly. Lower bits = less VRAM + faster memory-bound inference, more rounding error. You measure the curve.
**Where here.** The quantization sweep (Q4/Q8/fp16) and the quality/latency/VRAM curve.

### Reliable structured output on small models
**What.** Making a smaller model emit valid JSON / correct tool calls reliably.
**Why it exists.** Frontier models are trained to be excellent at tool-call formatting; smaller open-weight models are much flakier, and a malformed tool call breaks the agent loop. This is the #1 reason naive local agents fail.
**How it works (mechanism).** **Constrained decoding**: at each step, mask out tokens that would violate a grammar (GBNF) or JSON schema, so the sampled output is *guaranteed* well-formed — the model literally cannot emit invalid JSON. This is the same "constrained decoding" idea as Project 01's structured router, but now load-bearing because the base model is weaker. Add a validate-and-repair loop as a backstop.
**Where here.** Phase 2 — the mitigation that turns "flaky demo" into "works."

### Characterizing a tradeoff (the deliverable mindset)
**What.** Producing the data — quality, latency, cost, privacy — that a real on-prem-vs-hosted decision needs, rather than declaring a winner.
**Why it exists.** "Local is better/worse" is naive; the answer is "at this volume, with this privacy requirement, on this hardware, quantized to this level, here's the frontier." That analysis is exactly what an AI platform engineer produces for a build-vs-buy/deploy decision.
**Where here.** The hosted-vs-local benchmark + quantization curve + cost crossover.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Serving an open-weight model locally + local embeddings | Self-hosting basics for on-prem work |
| 1 | Porting an agent off a hosted API | Model-provider-agnostic agent design |
| 2 | Grammar-constrained decoding | The skill that makes small-model agents reliable |
| 3 | Offline RAG + small-context engineering | Working within real hardware limits |
| 4 | Quantization + hosted-vs-local benchmarking | The on-prem deployment analysis |
| 5 | A fully offline, dockerized stack | The artifact a privacy-sensitive employer wants to see |

## 4. Common Misconceptions & Mistakes

- **Prompt-and-hope JSON.** Small models need constrained decoding.
- **"Local = hosted quality."** Measure and report the gap; the value is privacy/cost.
- **Context overflow.** Trim retrieval/prompts for small context.
- **Unspecified quantization.** Report the level and the curve.
- **Not actually offline.** A stray hosted call breaks it; verify egress-blocked.

## 5. Understanding-check questions (with answer key)

**Q1 (Why local).** Give three concrete reasons an organization runs agents on local open-weight models despite frontier APIs being easier.
**A1.** (1) Privacy/compliance — data can't leave the network (healthcare/finance/air-gapped); (2) cost at scale — amortized hardware beats per-token pricing above some volume; (3) control/availability — pinned weights, no network dependency or vendor rate limits/outages.

**Q2 (Quantization).** What does going from fp16 to Q4 buy you and cost you? How do you choose the level?
**A2.** Buys: ~4× less VRAM and faster (memory-bound) inference, so it fits smaller hardware. Costs: rounding error → some quality loss. You choose by running the eval at 2–3 levels and reading the quality/latency/VRAM curve — pick the lowest precision that still clears your task's quality bar.

**Q3 (Structured output).** Why do small models fail at tool calls more, and how does constrained decoding fix it "by construction"?
**A3.** They're less reliably trained on strict formatting, so they drift into malformed JSON/wrong schemas. Constrained decoding masks any token that would violate the grammar/schema before sampling, so the output is guaranteed valid — the model can't emit invalid JSON, rather than being merely asked not to.

**Q4 (Tradeoff analysis).** Why is "local is better" the wrong conclusion? What's the right form of the answer?
**A4.** It ignores the axes that actually decide it. The right answer is a frontier: "at request volume V and privacy requirement P, on hardware H, quantized to Q, local matches hosted within X pp at Y× the latency and Z the cost — so choose local above volume V / when P forbids egress." A decision, backed by the curve.

**Q5 (Offline).** How do you *prove* the agent is offline, and why isn't "I used a local model" sufficient?
**A5.** Block network egress during a run and confirm it still completes (Project 07's egress test). "I used a local model" isn't sufficient because a stray hosted embedding call, telemetry, or fallback can silently reach the network — only an egress-blocked successful run proves true offline operation.
