# PROFESSOR-NOTES.md — Computer-Use / Browser Automation Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| Playwright basics (locators, actions, snapshots) | The agent drives a browser with it | Playwright Python docs → "Writing tests" + "Accessibility testing" |
| The accessibility (a11y) tree | Your primary grounding signal | MDN "Accessibility tree" overview; Playwright `accessibility.snapshot()` |
| Multimodal LLM calls (image + text) | The vision fallback + verifier | Your provider's vision API docs |
| Agent loops (plan/act/observe) | The control structure | Reuse Project 01's LangGraph loop understanding |

## 2. Core Concepts Taught

### Perception grounding (DOM/a11y vs. vision)
**What.** Turning "the login button" into a specific, actionable element on a live page.
**Why it exists.** Unlike API tools with typed interfaces, a web page is a messy, changing visual/structured hybrid. You need to *perceive* the page reliably enough to act. Two signals: the **accessibility tree** (structured — roles, names, handles; reliable and cheap) and **vision** (a screenshot — necessary for canvas/image-only UI; expensive and coordinate-fragile).
**How it works (mechanism).** Snapshot the a11y tree; match the intended target against element roles/names to get a stable locator; act via Playwright. When the a11y tree lacks the element (custom canvas, image button), fall back to sending the screenshot to a vision model to identify a region. A11y-first is the cost/reliability decision.
**Where here.** The Observer + Action Grounder.

### Action spaces
**What.** The finite set of things the agent can do (click/type/scroll/navigate/wait/done/ask_confirmation), each with a typed target.
**Why it exists.** A constrained, typed action space makes the agent's output verifiable and executable — free-form "do something" isn't actionable. It also lets you attach safety metadata (`reversible`) to each action.
**Where here.** The `Action`/`Step` types.

### Failure recovery / re-planning
**What.** Detecting that an action didn't achieve its sub-goal and re-planning, rather than blindly proceeding.
**Why it exists.** Real web tasks fail mid-way constantly (an element moved, a modal appeared, the page was slow). An agent that can't recover looks great in a scripted demo and dies live. Recovery — observe the unexpected state, re-plan — is what separates a demo from a usable agent.
**Where here.** The Progress Verifier → re-plan loop, with a step cap.

### Safety for irreversible actions
**What.** Structurally gating actions that can't be undone (pay, submit, delete, send) behind confirmation.
**Why it exists.** A computer-use agent acts on the real world; a wrong irreversible click has real consequences. Prompt-level "please confirm" isn't a boundary — the executor must *refuse* an irreversible step without an approval token (same principle as Project 04's approval gate and Project 05's read-only DB).
**Where here.** The Confirmation Gate.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Driving a browser + capturing page state | The substrate of every computer-use agent |
| 1 | Grounding intent to a real element | The core perception skill computer-use roles test |
| 2 | The plan/observe/act/verify loop | The web-agent control structure |
| 3 | Vision fallback for non-DOM UI | Handling the messy real web |
| 4 | Structural safety + recovery | What makes a computer-use agent deployable |
| 5 | Benchmarking task success | Quantifying a fuzzy capability |
| 6 | A live-run demo | The single most impressive artifact to show |

## 4. Common Misconceptions & Mistakes

- **Screenshot-only grounding.** Brittle, expensive, coordinate-fragile; a11y-first is better.
- **No step cap.** A stuck agent loops forever.
- **No irreversible-action gate.** Dangerous — a wrong click has real consequences.
- **Scoring on live sites.** Non-deterministic and risky; use controlled sites.
- **Ignoring recovery.** Scripted demos hide that real tasks fail mid-way.

## 5. Understanding-check questions (with answer key)

**Q1 (Grounding).** Why prefer the accessibility tree over screenshots for grounding? When must you fall back to vision?
**A1.** The a11y tree is structured (roles, names, stable handles), so grounding is reliable and cheap, and locators survive minor layout changes. Screenshots require the model to output coordinates, which are fragile and expensive. Fall back to vision when the target isn't in the a11y tree — canvas widgets, image-only buttons, or visually-encoded state.

**Q2 (Action space).** Why constrain actions to a typed set instead of free-form output? What does typing `reversible` buy you?
**A2.** A typed action space makes the agent's output executable and verifiable, and prevents nonsensical "actions." Tagging each action `reversible` lets you route irreversible ones through the confirmation gate structurally — safety metadata rides on the action itself.

**Q3 (Recovery).** Why does an agent that can't re-plan look good in a demo but fail live? Give a concrete failure it must recover from.
**A3.** Demos are scripted on a known-good path; live pages deviate. Concrete: after "click checkout," an unexpected "are you a member?" modal appears. A no-recovery agent proceeds against a stale plan (clicks where checkout *was*); a recovering agent observes the modal, re-plans (dismiss or answer it), then continues.

**Q4 (Safety).** Why is a prompt saying "confirm before buying" not a real safety boundary? What is?
**A4.** The model can ignore or be manipulated past a prompt instruction — it's advisory. A real boundary is structural: the executor refuses to perform any `Step` marked irreversible unless it carries an approval token issued by the confirmation gate, so an un-approved purchase is unrepresentable, not just discouraged.

**Q5 (Benchmark).** Why score on controlled sites instead of live public ones?
**A5.** Live sites are non-deterministic (content/layout changes, A/B tests), rate-limited, and governed by ToS — plus a purchase task would make real transactions. Controlled/sandbox sites give reproducible, safe, assertable goal states, so the success number is meaningful and re-runnable.
