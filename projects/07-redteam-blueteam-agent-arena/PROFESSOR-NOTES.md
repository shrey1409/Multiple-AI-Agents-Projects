# PROFESSOR-NOTES.md — Red-Team vs Blue-Team Agent Arena

**Framing note (read first).** Everything here is taught and built strictly as defensive security research inside an isolated lab against a deliberately vulnerable training app, using a fixed probe library. If you ever feel the urge to point any part of this at a system you don't own — stop. That's out of scope for this project and this portfolio.

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| OWASP Top 10 (web vuln classes) | You must recognize what the attacker probes for | OWASP Top 10 project page |
| Docker networking (`--internal`, bridge) | The lab's safety rests on network isolation | Docker "Networking overview" + "Network drivers" |
| Finite state machines | The referee is an FSM | Any short FSM primer |
| SOC metrics (MTTD/MTTR) | Your eval reports detection/response timing | Any intro to security operations metrics |

## 2. Core Concepts Taught

### FSM-constrained multi-agent handoffs
**What.** Restrict which agent acts next via an explicit transition table (allowed next-speakers per state), not an LLM deciding "whose turn".
**Why it exists.** In a free-form group chat, an LLM speaker-selector can be wrong, inconsistent, or — in an adversarial setting — manipulated. An FSM makes illegal transitions structurally impossible.
**How it works (mechanism).** Define states (`attacker_turn`, `defender_turn`) and a transition table; the referee consults it **in code** to pick the next actor. In AutoGen this is the group chat's allowed-transitions graph. Because it's code, no prompt injection into an agent can change whose turn it is.
**Where here.** The FSM Referee + the code-level out-of-turn test.

### Isolated evaluation environments ("the lab")
**What.** Run the system under test with no path to production/the broader network, so even a successful "attack" has zero blast radius.
**Why it exists.** Security research is only safe when failure is contained — standard practice for CTFs, internal red teams, bug-bounty test envs.
**How it works.** An `--internal` Docker network with no default route out, **verified** (egress test fails) before every session — not assumed.
**Where here.** The target's network config — the single most important design decision, ahead of any agent logic.

### Defensive-research vs. offensive-tool framing
**What.** The same technical work (probing for vulns) can be scoped/presented as "an attack tool" or "an evaluation harness measuring detection/response" — the framing changes both safety posture and hiring signal.
**Why it exists.** Security-adjacent roles value defensive maturity (can you measure and improve detection/response) over raw offensive capability; a measurement-framed project is safer to build and a stronger signal.
**Where here.** The entire eval is built around defender MTTD/MTTP + coverage, not "look what the attacker found."

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Verifying (not assuming) isolation | The most transferable security-engineering habit here |
| 1 | Structural constraints over LLM judgment | Portfolio-wide theme: verify safety in code, not prompts |
| 2 | Probing known vuln classes in a scoped, cataloged way | Automated security testing for security-adjacent AI roles |
| 3 | Log-analysis detection + automated patch proposals | Maps to real SOC automation |
| 4 | Designing security-exercise metrics (coverage, MTTD, MTTP) | Quantifying security posture |
| 5 | Framing a sensitive project responsibly | How you write about it affects reception as much as what you built |

## 4. Common Misconceptions & Mistakes

- **"Build a hacking bot."** The differentiator is the defender crew + timing metrics; an attacker-only version is less safe and less interesting.
- **Assuming Docker default networking is isolated.** Default bridge often reaches the internet; use `--internal` and verify.
- **FSM "mostly enforced" via prompting.** Advisory ≠ enforced; it's not the FSM pattern then.
- **Auto-applying patches.** Removes the human checkpoint for no demo benefit.
- **Offensive-showcase README tone.** Undermines safety framing and hiring signal.

## 5. Understanding-check questions (with answer key)

**Q1 (FSM handoffs).** Why is an LLM "whose turn is it" decision insufficient here, when it might be fine in a normal chat? What's different?
**A1.** In a normal chat, a wrong speaker choice is a minor UX glitch. Here the context is adversarial — a probe result or a crafted log line could influence an LLM speaker-selector to, say, let the attacker act twice. An FSM removes the LLM from that decision entirely, so manipulation can't reorder turns.

**Q2 (Isolation).** Describe the specific test that *verifies* no outbound route. What does false isolation look like?
**A2.** From inside the target's network namespace, attempt to reach an external host (`curl https://example.com` or ping a public IP) and assert it fails/times out. False isolation: the container is on a default bridge that NATs to the internet, so the egress test unexpectedly succeeds — meaning a real exploit could reach outside.

**Q3 (Framing).** Rewrite "My agent successfully hacked the login form using SQL injection" in defensive-research framing.
**A3.** "In the isolated lab, the attacker crew triggered the known SQL-injection challenge on the login form; the defender crew detected the injection signature in the request log within 8s (MTTD) and proposed an input-validation patch within 40s (MTTP)." It shifts the headline to measured detection/response.

**Q4 (Defender).** Why proposal-only, not auto-apply? What's lost safety-wise if you remove that boundary even in the sandbox?
**A4.** Proposal-only keeps a human in the loop for any change, so a wrong "fix" can't silently alter the running system. Removing it trains the reflex to let agents mutate systems autonomously — the exact habit you don't want in security work, and it can mask whether the detection was even correct.

**Q5 (Scoring).** Why report the distribution of detection times, not just the mean?
**A5.** A single mean hides tail behavior — e.g., the defender catches most attacks in 5s but misses one class for 5 minutes. In security, the slow tail is often what matters most; the distribution surfaces it.
