# PROFESSOR-NOTES.md — Voice / Real-Time Conversational Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| STT and TTS basics (streaming vs. batch) | The pipeline's ends | Whisper README; your TTS provider's streaming docs |
| Voice Activity Detection (VAD) | Detects when the user is speaking | Any VAD primer (e.g., Silero VAD README) |
| Async/streaming programming | The whole loop is concurrent streams | Python `asyncio` streams |
| A portfolio agent with a Contract API | It's the "brain" | Project 01 or 04 |

## 2. Core Concepts Taught

### The real-time latency budget
**What.** The end-to-end time from when the user stops speaking to when the agent starts speaking, decomposed into stages (turn detection + STT finalize + brain think + TTS first audio), each with a sub-budget.
**Why it exists.** Humans perceive >~800ms of silence as awkward; a voice agent that's correct but slow feels broken. Unlike every other portfolio project (second-scale HTTP), voice has a *human-perceptible* budget you must engineer to.
**How it works (mechanism).** You attack each stage: streaming STT finalizes fast because most of the transcript is already done when the user stops; the brain **streams** so you don't wait for the full response; TTS streams so first audio plays while the rest synthesizes. The decisive trick is **pipelining** — overlapping stages instead of running them in sequence.
**Where here.** The §2 latency-budget table — the core engineering artifact.

### Pipelining vs. sequencing
**What.** Overlapping the STT → brain → TTS stages so they run concurrently on a rolling window, rather than one-after-another.
**Why it exists.** Sequential latency is the *sum* of stages (easily 3s+). Pipelined latency is closer to the *longest single stage plus first-chunk times*, because you start speaking sentence 1 while generating sentence 2, and start reasoning on partial transcripts.
**Where here.** The design that gets P50 under 1s.

### Turn-taking and turn detection
**What.** Deciding when the user has actually finished a turn (vs. a thinking pause) so the agent responds at the right moment.
**Why it exists.** A bare silence timer either cuts users off (timer too short — they paused to think) or feels laggy (timer too long). Semantic turn detection asks "is this a complete thought?" to decide, which is what makes conversation feel natural.
**Where here.** The VAD + semantic turn-detection stage and the false-cutoff metric.

### Barge-in (interruption handling)
**What.** Letting the user interrupt the agent mid-sentence; the agent stops and listens.
**Why it exists.** Real conversation is full of interruptions ("no, I meant..."). An agent that plows through its full response while the user is talking is the #1 tell of a robotic voice bot.
**How it works (mechanism).** Keep the mic open while speaking; on detected user speech, **cancel** the in-flight TTS playback *and* the streaming brain call (a cancellable handle), flush, and treat the new speech as a fresh turn.
**Where here.** The Interrupt controller.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Audio I/O + VAD + STT/TTS round-trip | The substrate of any voice agent |
| 1 | Reusing a text agent as a voice brain | Decoupling reasoning from modality |
| 2 | Streaming + pipelining for latency | The core real-time-systems skill voice roles test |
| 3 | Interruption handling | What separates natural from robotic |
| 4 | Semantic turn detection | The subtle skill behind good conversation UX |
| 5 | Measuring a voice agent honestly | Latency/WER/barge-in as real metrics |

## 4. Common Misconceptions & Mistakes

- **Sequential pipeline.** Sums the latencies; pipeline instead.
- **No barge-in.** Talking over the user; the classic tell.
- **Silence-timer-only turn detection.** Cuts thinking pauses or lags.
- **Ignoring the budget.** A slow-but-correct voice agent is unusable.
- **Rebuilding the brain.** Reuse a portfolio agent; the loop is the point.

## 5. Understanding-check questions (with answer key)

**Q1 (Latency budget).** Why does voice have a stricter latency requirement than your text agents, and what's the perceptual threshold?
**A1.** In text, users tolerate seconds of "thinking"; in voice, silence is dead air and humans perceive >~800ms as awkward or broken. So the end-to-end budget is roughly sub-second, far tighter than the second-scale budgets elsewhere in the portfolio.

**Q2 (Pipelining).** How does pipelining get end-to-end latency below the sum of the stages? Give the key overlap.
**A2.** By overlapping stages instead of sequencing. The decisive overlap: start TTS on the first complete sentence while the LLM is still generating the rest, and feed streaming STT partials into context before the user even finishes — so total time approaches the longest stage plus first-chunk times, not the sum.

**Q3 (Turn detection).** Why is a fixed silence timeout a bad turn detector? What does semantic turn detection add?
**A3.** Too short a timeout cuts users off during a thinking pause; too long adds lag. Semantic turn detection judges whether the utterance is a *complete thought*, so it waits through mid-sentence pauses but responds promptly at a real end-of-turn — decoupling "finished talking" from "went silent for N ms."

**Q4 (Barge-in).** What two things must the interrupt controller cancel, and what breaks if it cancels only one?
**A4.** It must cancel both the TTS playback and the in-flight (streaming) brain call. Cancel only the TTS and the brain keeps generating a now-irrelevant answer (wasted cost, and it may resume speaking); cancel only the brain and the agent keeps *speaking* the old response over the user. Both must stop for a clean interruption.

**Q5 (Reuse).** Why reuse a portfolio agent as the brain instead of building a new one?
**A5.** The learning objective is the real-time voice loop (budget, pipelining, barge-in, turn detection), not new reasoning. Reusing Project 01/04 over its Contract API keeps scope on the voice engineering and demonstrates that your reasoning agents are modality-agnostic behind a clean interface.
