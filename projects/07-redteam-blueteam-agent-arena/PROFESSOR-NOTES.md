# PROFESSOR-NOTES.md — Red-Team vs Blue-Team Agent Arena

**Framing note (read first):** everything in this project is taught and built strictly as defensive security research inside an isolated lab against a deliberately vulnerable training application. If you ever feel the urge to point any part of this at a system you don't own, stop — that's out of scope for this project and this portfolio.

## 1. Prerequisites

| Concept | Why you need it first | Best specific resource |
|---|---|---|
| OWASP Top 10 (conceptual understanding of common web vulnerability classes) | You need to recognize what the attacker crew is actually probing for | OWASP's own Top 10 project page |
| Docker networking basics (bridge networks, `--internal` flag) | The lab's safety property depends entirely on correct network isolation | Docker's own networking docs |
| Finite state machines (states, transitions, what makes a transition "illegal") | The referee's entire design is an FSM | Any short FSM primer — you likely already know this from a CS fundamentals course |

## 2. Core Concepts Taught

### Finite-state-machine-constrained multi-agent handoffs
**What:** restricting which agent can act next using an explicit state-transition table (a graph of allowed next-speakers per current state), rather than letting an LLM decide "whose turn is it" each round.
**Why it exists:** in a free-form multi-agent group chat, an LLM-mediated speaker-selection function can be wrong, inconsistent, or (in an adversarial context like this project) manipulated — an FSM makes illegal transitions structurally impossible rather than merely discouraged.
**How it works:** define states (e.g., `attacker_turn`, `defender_turn`) and a transition table of which states can follow which; the referee consults this table in code, not via an LLM call, to decide who acts next.
**Where it's used here:** the FSM Referee, and specifically the code-level test in the Definition of Done that verifies an out-of-turn action is actually rejected.

### Isolated evaluation environments ("the lab")
**What:** running a system under test in an environment with no path to production systems or the broader network, so that even a successful "attack" has zero real-world blast radius.
**Why it exists:** security research and red-team exercises are only safe to run when failure is contained; this is the standard practice at every organization that does this kind of testing professionally (CTF competitions, internal red teams, bug bounty test environments).
**How it works:** an internal-only Docker network with no default route out, verified (not assumed) before every session.
**Where it's used here:** the target application's network configuration — treated as the single most important design decision in the whole project, ahead of any agent logic.

### Defensive-research framing vs. offensive-tool framing
**What:** the same underlying technical work (probing for vulnerabilities) can be presented and scoped either as "building an attack tool" or as "building an evaluation harness that measures detection/response," and the framing changes both the safety posture and how the project reads to a hiring audience.
**Why it exists:** security-adjacent roles overwhelmingly value evidence of defensive maturity (can you measure and improve detection/response) over evidence of offensive capability alone; a project framed around measurement (time-to-detect, time-to-patch) is both the safer thing to build and the stronger portfolio signal.
**Where it's used here:** the entire eval strategy (§6) is built around timing and coverage metrics for the *defender*, not just "look what the attacker found."

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Why it matters for your career |
|---|---|---|
| 0 (Lab Setup) | Verifying (not assuming) network isolation | The single most transferable "security engineering mindset" habit in this project |
| 1 (FSM Referee) | Structural constraints over LLM-mediated judgment calls | Reinforces a theme across this whole portfolio: verify safety properties in code, not prompts |
| 2 (Attacker Crew) | Recognizing and probing for known vulnerability classes in a scoped, catalogued way | Directly relevant to security-adjacent AI-engineering roles doing automated testing |
| 3 (Defender Crew) | Log-analysis-driven detection, automated patch proposal generation | Maps to real SOC (security operations center) automation work |
| 4 (Scoring) | Designing metrics for a security exercise (coverage, time-to-detect, time-to-patch) | Teaches you to measure security posture quantitatively, not just qualitatively |
| 5 (Writeup) | Framing a sensitive-sounding project responsibly for a hiring audience | A specific, learnable skill — how you write about this project affects how it's received as much as what you built |

## 4. Common Misconceptions & Mistakes

- **Treating this as "build a hacking bot."** The actual differentiator is the defender crew and the timing metrics — an attacker-only version of this project is both less safe and less interesting.
- **Assuming Docker's default networking is isolated enough.** Default bridge networks can often still reach the internet; explicit `--internal` (or equivalent) configuration and verification are required, not assumed.
- **Letting the FSM be "mostly enforced" via prompting.** If the transition table is advisory rather than a hard code-level check, you haven't actually built the FSM pattern — you've built a group chat with extra words in the system prompt.
- **Auto-applying the defender's patches.** Removes the human checkpoint for no real benefit to the demo, and is inconsistent with the HITL principle used throughout this portfolio.
- **Writing the README in an offensive-showcase tone.** Undermines both safety framing and hiring signal — lead with scope and safety, then present detection/response metrics as the headline result.

## Understanding-check questions

**After §2 (FSM-constrained handoffs):** Why is an LLM-mediated "whose turn is it" decision insufficient for this specific project, when it might be fine in a normal multi-agent chat? What's different about the adversarial context?

**After §2 (Isolated evaluation environments):** Describe the specific test you'd run to *verify* (not assume) that the target has no outbound network route. What would a false sense of isolation look like?

**After §2 (Defensive-research framing):** Rewrite this sentence in defensive-research framing: "My agent successfully hacked the login form using SQL injection." What changed, and why does it matter for a hiring audience?

**After Phase 3 (Defender Crew):** Why does the patch-proposal agent only produce a proposal rather than applying the fix? What would you lose, safety-wise, if you removed that boundary — even inside the sandbox?

**After Phase 4 (Scoring):** Why report the distribution of detection times (not just the mean) for a security exercise? What could a single average number hide that matters for this narrative?
