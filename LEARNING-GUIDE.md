# LEARNING-GUIDE.md — The AI-Agent Portfolio, Explained in Build Order

This is the companion to `ROADMAP.md`. The ROADMAP is the *sequence*; this is the *why* — for every one of the 18 projects, in the exact order you should build them: what the system actually does, who runs systems like it in production, what you learn, and where those skills take your career. Read a project's section here before you open its `projects/<NN>/PLAN.md`.

**How to read each entry:** (1) *What it is* — plain English. (2) *Real-world use case* — who runs this, what business problem, which role builds it. (3) *Skills learned* — concrete. (4) *Where these skills go next* — roles, interview questions, future systems, resume phrasing. (5) *Builds on / feeds into* — the dependency wiring.

A skills-coverage matrix is at the end so you can see coverage and gaps at a glance.

> **India-first note.** These projects are localized for the Indian market (NSE/BSE, GST/KYC, DPDP Act, INR, Indian-language models) by default, with a pluggable data layer that swaps to global markets by config — see each project's PLAN.md §10 "Localization" and Project 01's "worldwide mode" adapter lesson. **The technical curriculum is identical** to the global version; only data sources, document types, regulators, currency, and example companies change. The real-world-use-case notes below name Indian companies/fintechs (Zerodha, Groww, PhonePe, Razorpay, Cred, Sarvam, etc.) where they run systems like these, alongside global players, so you can see who hires for each skill *in India* first.

---

## 1 · Project 01 — Multi-Agent Financial Research Analyst
*(foundation — build this first)*

**What it is.** You give it a stock ticker; it returns a cited, investment-research-style brief. Under the hood a *supervisor* agent routes work to four specialists — market data, news sentiment, fundamentals (via Corrective RAG over SEC filings), and quantitative ratios — then a *critic* agent reviews the draft and forces revisions before it ships. It's a complete multi-agent system: routing, retrieval, tool use, a self-correction loop, an eval harness, and a live deployment.

It's the foundation because almost every pattern the rest of the portfolio uses appears here first — orchestration, RAG, reflection, structured tool calls, and a deployed API. It also emits the **Target Agent Contract** (a structured trajectory in its response) that the observability, eval, guardrail, and release-gate projects all consume.

**Real-world use case.** This is what an **applied-AI / AI platform engineer at a fintech, asset manager, or research shop** builds: analyst-augmentation tools that gather multi-source data and draft grounded, cited summaries (Bloomberg-style research assistants, robo-advisory research, equity-research copilots). **In India specifically**, this is the research-copilot shape behind **Zerodha** (and its Varsity/Kite ecosystem), **Groww**, **smallcase**, **Tickertape/Screener.in**, **Angel One**, and **Upstox** — retail-broking and wealth platforms that increasingly ship "explain this stock" assistants over NSE/BSE data and Indian filings. The same supervisor-plus-specialists shape underlies enterprise "research assistants" in consulting, insurance underwriting, and market intelligence. The business problem: analyst time is expensive and Indian retail investors want grounded, jargon-free research on NSE/BSE names — automatable if, and only if, it's grounded and citable (promoter-pledge and FII/DII context included).

**Skills learned.** LangGraph state machines with cycles; the supervisor multi-agent pattern; structured output / constrained decoding for reliable routing; Corrective RAG (grade retrieval, fall back to web); reflection loops with a bounded revision cap; tool design and sandboxed code execution; section-aware document chunking; an eval harness with an LLM judge and a citation-integrity check; containerization and deployment.

**Where these skills go next.** Multi-agent orchestration is *the* most-cited agent-engineering skill in 2026 job posts — this is the project that lets you claim it. Interview questions it prepares you for: "How would you design a multi-agent system?", "How do you stop a reflection loop from running forever?", "How do you prevent an agent from hallucinating a citation?", "When do you use a router vs. a ReAct loop?" It unlocks every larger system that coordinates specialists (research platforms, agentic workflows, planning systems). Resume phrasing: *"Built a production multi-agent research system (LangGraph supervisor + Corrective RAG + reflection) with a 15-case eval harness; 85% of outputs scored ≥4/5 on grounding, P95 <90s, <$0.25/report."*

**Builds on / feeds into.** Builds on: nothing (start here). Feeds into: **13** (instruments it), **10** (reuses its corpus), **03** (evaluates it), **08** (rebuilds its specialists), **11** (protects it), **16** (reuses its corpus), **17/18** (reuse it as a brain / port target).

---

## 2 · Project 13 — Agent Observability Stack
*(see inside the agent you just built)*

**What it is.** You instrument Project 01 with OpenTelemetry so that every run emits a *trace*: a tree of spans (the run → each graph node → each tool call) carrying model, token counts, latency, and cost, following the OpenTelemetry **GenAI semantic conventions**. On top of the traces you build a viewer and cost/latency/error dashboards, and you ship a small reusable wrapper (`agent-otel`) other agents adopt by decorating their nodes. This project also *defines* the Target Agent Contract that later projects depend on — it's the reference way an agent emits its own trajectory.

Observability answers a different question than evaluation: not "was the output good?" but "what did the agent actually *do*, and where did the time/tokens/errors go?" When an agent is slow, expensive, or wrong, the trace tells you *where*.

**Real-world use case.** This is what an **AI platform / LLMOps engineer** builds and what products like Langfuse, Arize Phoenix, LangSmith, and Datadog's LLM observability sell. Every company running agents in production needs it — you cannot operate what you cannot see. The business problem: an un-instrumented agent that costs 3× more than expected or times out intermittently is a black box you can't debug or optimize. Teams in fintech, healthcare, and SaaS stand up exactly this before trusting an agent with real traffic.

**Skills learned.** Distributed tracing (spans, traces, parent/child context); OpenTelemetry SDK + OTLP + Collector; the GenAI semantic conventions (a portable schema, not a bespoke logger); per-span cost attribution; building trace viewers and cost/latency dashboards; designing an observability *contract* other systems build on; attribute hygiene (redaction — never log PII/secrets in spans).

**Where these skills go next.** LLMOps/observability is a fast-growing specialization; "I instrument agents with OTel GenAI conventions" is a specific, checkable claim most candidates can't make. Interview questions: "How do you debug a slow/expensive agent?", "What would you log for an agent in production?", "Build vs. buy for LLM observability?" Unlocks: SRE/platform work on agent fleets, cost-optimization mandates, and the instrumentation every eval/guardrail system relies on. Resume phrasing: *"Built an OpenTelemetry-based agent observability stack (GenAI semantic conventions, per-span cost attribution, <5% overhead) and defined the trajectory contract consumed by our eval and CI systems."*

**Builds on / feeds into.** Builds on: **01** (instruments it). Feeds into: **03, 11, 12, 17** (all consume the Target Agent Contract it implements).

---

## 3 · Project 10 — RAG Architecture Bake-Off
*(go deep on retrieval)*

**What it is.** You implement all four LangGraph RAG variants — Agentic, Adaptive, Corrective, and Self-RAG — plus a naive baseline, over one shared corpus and one shared, categorized question set, then benchmark them head-to-head with RAGAS (faithfulness, relevancy, context precision/recall) plus latency and cost. The output is a data-backed "use variant X when Y" recommendation, not four disconnected demos.

The real lesson is *controlled experimentation*: the whole thing is only valid because you freeze the corpus, chunking, embeddings, and index and vary only the retrieval control-flow. That discipline — and the ability to say which RAG architecture to use and why, with numbers — is rarer than knowing any single variant.

**Real-world use case.** This is what an **applied scientist / RAG engineer** does when a company is choosing how to build retrieval for a knowledge-base assistant, support bot, or search product. Every org shipping RAG (which in 2026 is most of them) faces "which retrieval strategy for our data?" — and usually answers it by vibes. Doing it as a measured experiment is what distinguishes a serious team. Industries: enterprise search, customer support, legal/financial knowledge tools, healthcare literature assistants.

**Skills learned.** The four RAG variants and precisely what each mechanism buys over naive; controlled-experiment design (isolating confounds); RAGAS metrics and what they actually compute; recognizing and avoiding the Agentic-vs-Adaptive collapse; reporting category-level results (not misleading aggregates); statistical honesty on small samples.

**Where these skills go next.** "Agentic RAG" is one of the hottest 2026 keywords; this project lets you speak to it with evidence. Interview questions: "What's the difference between Corrective and Self-RAG?", "How would you evaluate a RAG system?", "Your RAG hallucinates — how do you diagnose it?" Unlocks: any advanced retrieval work, and the experimental rigor transfers to model selection and prompt A/B testing. Resume phrasing: *"Ran a controlled bake-off of four RAG architectures on a shared corpus/question set (RAGAS + cost/latency), producing a data-backed retrieval-strategy recommendation."*

**Builds on / feeds into.** Builds on: **01** (RAG concepts + reusable corpus). Feeds into: **16** (multimodal RAG extends this), and the retrieval judgment reused everywhere.

---

## 4 · Project 03 — Agent Evaluation & Observability Platform
*(prove it works — the rarest skill)*

**What it is.** A harness that stress-tests *other* agents. Simulated-user agents (scripted and adversarial) drive a target agent through many conversations; an LLM-as-judge grades every trajectory against a rubric; an aggregator detects regressions between agent versions with a real statistical test; a dashboard and a CI gate surface the results. Crucially, you *calibrate* the judge against hand-labeled examples (Cohen's κ) and *prove* the regression detector works by injecting deliberately-broken agent versions and confirming it catches all of them.

Most candidates can build an agent; almost none can prove one works or catch a regression. This is the project that makes you the person a team trusts to put an agent into production.

**Real-world use case.** This is what an **AI quality / evals engineer** (an increasingly named role at OpenAI, Anthropic, Scale, and every serious AI product team) does full-time. The business problem: LLM agents are non-deterministic, so "it worked when I tried it" is worthless — you need a repeatable, calibrated way to measure quality and block regressions before they ship. Fintech, healthcare, and any regulated or high-stakes deployment cannot ship agents without this.

**Skills learned.** LLM-as-judge design and its biases (position/verbosity/self-enhancement); judge calibration with Cohen's κ (chance-corrected agreement); simulated-user and adversarial-persona testing; trajectory (process) vs. outcome evaluation; statistical regression detection for non-deterministic systems (two-proportion tests, confidence intervals, tolerance bands); wiring evals into CI.

**Where these skills go next.** Evals is the single most credibility-generating skill in this portfolio. Interview questions: "How do you evaluate an agent?", "How do you know your LLM judge is trustworthy?", "How do you catch a regression in a non-deterministic system?" Unlocks the "evals engineer" career track and eval-driven development. Resume phrasing: *"Built an agent-evaluation platform (calibrated LLM-as-judge at κ≥0.6, adversarial simulated users, statistical regression detection) that caught 3/3 injected regressions and gates CI."*

**Builds on / feeds into.** Builds on: **01** (the target), **13** (consumes its trajectory contract). Feeds into: **12** (packages this harness as a CI product).

> ✅ **Resume Checkpoint 1 — first shippable portfolio.** With 01 + 13 + 03 you have a deployed multi-agent system, full observability, and a calibrated eval harness. Start applying now.

---

## 5 · Project 09 — MCP Server Trilogy
*(author the protocol, don't just consume it)*

**What it is.** You build, test, document, and *publish* three standalone MCP (Model Context Protocol) servers — an API wrapper, a knowledge-store, and a task-tracker — each passing MCP Inspector's compliance checks, each with a real test suite, and at least one published to PyPI and submitted to the official MCP Registry. The emphasis is the *provider* side: designing what to expose, how to type it, how to fail, and proving it works from a client you didn't write.

Most tutorials teach you to *call* tools; almost nobody builds standards-compliant, publishable tool servers. That authoring skill — closer to API design than prompt engineering — is rarer and more directly hireable.

**Real-world use case.** This is what a **platform / developer-tools engineer** does when a company exposes its internal systems to AI agents. MCP has become the standard integration layer (adopted across Anthropic, OpenAI, and the broader ecosystem), so "build an MCP server for our internal API/database/knowledge base" is a real, common task. Every company that wants its agents to safely reach internal tools needs people who can author compliant servers.

**Skills learned.** MCP authoring (tools/resources/prompts, capability negotiation, transports — stdio + Streamable HTTP); protocol compliance vs. "works for me"; idempotency and error semantics as API design; testing at the protocol/transport level (subprocess, not internal functions); packaging + publishing (PyPI + the official registry's `server.json`).

**Where these skills go next.** "I've authored and published MCP servers" is a concrete, checkable claim on a resume and in interviews. Interview questions: "What's MCP and why does it matter?", "Tool vs. resource — when each?", "How do you make a tool idempotent?" Unlocks integration/platform roles and is the foundation for the A2A project. Resume phrasing: *"Authored and published three spec-compliant MCP servers (MCP Inspector-verified, ≥80% tool test coverage, one live on PyPI + the official MCP Registry)."*

**Builds on / feeds into.** Builds on: nothing hard (self-contained). Feeds into: **04** (consumes your servers), **08** (each A2A agent has internal MCP tools).

---

## 6 · Project 04 — MCP-Native Personal Ops Agent
*(become your own first consumer + add proactivity)*

**What it is.** A persistent personal assistant whose tools are *exclusively* MCP servers you wrote (calendar, GitHub, notes). It keeps long-term memory of your preferences, runs scheduled proactive checks ("3 PRs need review, interview tomorrow"), and requires explicit approval before any externally-visible action. You are now the *client* side of the protocol you authored in Project 09, and you add proactivity (scheduled triggers) and a first taste of memory.

**Real-world use case.** This is what teams building **AI assistants / copilots** (personal productivity, internal ops bots, "agentic" SaaS features) build. The proactive-plus-approval pattern — an agent that initiates on a schedule but gates consequential actions behind a human — is exactly how responsible production assistants work (think enterprise scheduling assistants, ops-automation bots, DevRel triage agents). Business problem: reactive chatbots are low-value; a proactive, memory-backed assistant that's still safe is genuinely useful.

**Skills learned.** MCP *client* integration; long-term memory basics (semantic recall of preferences); proactive/scheduled agents and precision tuning (false-positive rate); structural approval gates (an action is un-executable without an approval token); the tools-vs-resources distinction from the consumer side.

**Where these skills go next.** Assistant/copilot roles are everywhere; this demonstrates the full MCP round-trip plus proactivity. Interview questions: "How does an agent decide when to act on its own?", "How do you keep a proactive agent from being a notification firehose?", "How do you gate an agent's actions?" Unlocks the memory system (14) and the voice agent (17), both of which reuse this as a brain. Resume phrasing: *"Built a proactive, memory-backed personal-ops agent consuming self-authored MCP servers, with a structurally-enforced human-approval gate (0 unapproved external actions over a 2-week trial)."*

**Builds on / feeds into.** Builds on: **09** (its servers). Feeds into: **14** (memory system plugs in here), **17** (voice reuses this brain).

---

## 7 · Project 14 — Long-Term Memory System
*(make memory a real system, not a vector dump)*

**What it is.** A standalone memory service implementing three memory types — *episodic* (time-stamped events), *semantic* (distilled facts/preferences), and *procedural* (learned how-tos) — with explicit policies: a write policy (what's worth storing), consolidation (distilling repeated episodes into stable facts), forgetting/decay, and supersession (handling "I changed my mind" without losing history). You benchmark it against a naive "embed everything" baseline on a multi-session recall task, and plug it into Project 04.

The point is that "embed every message and retrieve top-k" is a *retrieval* system, not a *memory* system — the engineering is in the policies, and the credibility is in benchmarking that they beat the naive approach.

**Real-world use case.** This is what teams building **stateful agents / personalized assistants** build (mem0, Letta/MemGPT, Zep are companies selling exactly this). The business problem: agents that start from zero every session feel dumb and can't personalize; but naive memory grows unbounded, returns stale contradicted facts, and never actually "learns." Anyone building a long-lived assistant (customer relationship agents, tutoring, healthcare companions, coding copilots with project memory) needs real memory engineering.

**Skills learned.** The episodic/semantic/procedural taxonomy; memory write/consolidation/forgetting/supersession policies; contradiction handling (supersede, don't delete); type-aware bounded retrieval under a token budget; benchmarking memory against a baseline (the part most memory demos skip).

**Where these skills go next.** "Stateful agents / agent memory" is a named 2026 hiring signal. Interview questions: "How would you give an agent long-term memory?", "How do you handle a user changing a stated preference?", "Why isn't a vector store alone 'memory'?" Unlocks personalization systems and long-horizon agents. Resume phrasing: *"Built a long-term memory service (episodic/semantic/procedural with consolidation and supersession) that beat a naive-embedding baseline by 15+ points on multi-session recall at 70% fewer context tokens."*

**Builds on / feeds into.** Builds on: **04** (plugs into it as the reuse proof). Feeds into: personalization for any later agent.

---

## 8 · Project 08 — A2A Multi-Framework Agent Network
*(interoperability across frameworks)*

**What it is.** You rebuild three of Project 01's specialists as *independent services in three different frameworks* — LangGraph, CrewAI, and Agno — each exposing itself over Google's Agent-to-Agent (A2A) protocol, each with internal MCP tools, all called by a discovery-based coordinator. The headline proof: you swap one specialist's framework and the coordinator needs *zero* code changes. This demonstrates interoperability, not just single-framework proficiency, and exercises MCP and A2A together — the "2026 protocol stack."

**Real-world use case.** This is what an **agent-platform architect** designs when an organization has agents built by different teams in different frameworks (the normal state of any large company) that must cooperate. A2A is the emerging standard for cross-framework, cross-team agent delegation. Business problem: no single team builds every agent in the same framework, and an orchestrator hardwired to one framework is brittle exactly where production can't afford it. Relevant to enterprise "agent ecosystems," multi-vendor AI platforms, and B2B agent marketplaces.

**Skills learned.** The A2A protocol (agent cards, `/.well-known/agent-card.json` discovery, JSON-RPC `message/send`, task lifecycle); framework-agnostic orchestration; proving an abstraction boundary by exercising it (the swap); using MCP (agent→tool) and A2A (agent→agent) together and explaining the difference cold; multi-framework fluency (LangGraph + CrewAI + Agno).

**Where these skills go next.** Protocol literacy (MCP + A2A) is a specific, rare 2026 signal. Interview questions: "MCP vs. A2A?", "How do agents from different frameworks cooperate?", "How do you make an orchestrator framework-agnostic?" Unlocks agent-platform/architecture roles. Resume phrasing: *"Built a framework-agnostic agent network over Google A2A (LangGraph + CrewAI + Agno specialists, MCP-wrapped tools); proved interop by swapping an agent's framework with zero coordinator changes."*

**Builds on / feeds into.** Builds on: **01** (specialists to port), **04/09** (MCP). Feeds into: agent-platform architecture work.

> ✅ **Resume Checkpoint 2 — protocol-fluent.** With 09 + 04 + 14 + 08 you demonstrate MCP authoring, MCP consumption, memory, and A2A interop — the protocol stack most candidates lack.

---

## 9 · Project 02 — Document-Processing HITL Pipeline
*(the durable, auditable enterprise pattern)*

**What it is.** A pipeline that watches an inbox, classifies documents (invoices/claims/contracts), extracts structured fields, validates against *deterministic* business rules, and — for anything risky or low-confidence — pauses via LangGraph's `interrupt()` and asks a human for approval (Slack) before executing an action. Every action is idempotent and every step is logged to an immutable audit trail. The hard, valuable parts are durable execution (a paused workflow survives a process restart and resumes exactly), idempotency (retries never double-act), and auditability.

**Real-world use case.** This is the single most common real-world enterprise agent pattern, and it's what a **backend/AI engineer at a fintech, insurer, healthcare, or ops-automation company** builds. **In India**, this is the GST-invoice / **KYC** / claims-processing automation behind **Razorpay** and **PhonePe** (merchant onboarding + KYC at scale), **Cred**, **Zerodha/Groww** account-opening, and every NBFC/insurer doing IRDAI-regulated claims — all under the **DPDP Act 2023**, which makes the human-approval gate and audit trail a *compliance control*, not just good practice (masking Aadhaar to last-4, data minimization). Business problem: high-volume document workflows are automatable, but only if consequential actions are human-gated, auditable, and safe under retries — exactly where AI agents generate ROI in regulated Indian industries today.

**Skills learned.** Durable execution / human-in-the-loop with interrupt-and-resume (survives restarts via a Postgres checkpointer); idempotency and at-least-once semantics (exactly-once is a myth); audit logging as a design constraint; deterministic rules engines vs. LLM validation (and *when not* to use an LLM); real confidence estimation (self-consistency, not LLM self-report); confidence-vs-risk routing.

**Where these skills go next.** This is the "I can ship agents in a regulated enterprise" project. Interview questions: "How does an approval workflow survive a server restart?", "Why is exactly-once impossible and what do you do instead?", "Why use rules instead of an LLM to validate?" Unlocks enterprise-automation and backend-heavy AI roles. Resume phrasing: *"Built a durable human-in-the-loop document pipeline (LangGraph interrupt/resume, idempotent actions, full audit trail); 100% HITL recall on high-risk items, resumes correctly across process restarts."*

**Builds on / feeds into.** Builds on: nothing hard (self-contained; reuses HITL ideas from 04). Feeds into: **11** (protects it), **12** (second target repo).

---

## 10 · Project 11 — Agent Guardrail Gateway
*(reusable safety middleware)*

**What it is.** A standalone middleware/proxy that sits in front of *any* agent and enforces four protections: PII redaction (inbound + outbound), prompt-injection detection, spend/rate limits, and an action allow-list with human escalation. You red-team it with an adversarial suite and report block rates *and* false-positive rates. The key framing: it's agent-agnostic — "I built a gateway that protects any agent" is a far stronger claim than "I added a safety check to my one agent."

**Real-world use case.** This is what an **AI security / trust-and-safety / platform engineer** builds — the agent equivalent of a web-application firewall (companies like Lakera, Robust Intelligence, and every enterprise AI-platform team build exactly this). **In India**, this is what any fintech deploying agents on Aadhaar/PAN/UPI data needs to satisfy the **DPDP Act** as a data-processor control — the Indian PII detectors here (Aadhaar with its Verhoeff checksum, PAN, UPI VPA, IFSC, GSTIN) are a *stronger* deterministic-validation showcase than the US SSN/card case. Business problem: agents leak PII into logs, get prompt-injected, and can spend unboundedly; a reusable safety layer is defense-in-depth that lets an org (Razorpay/PhonePe/Cred-scale) deploy many agents safely and compliantly.

**Skills learned.** The middleware/gateway safety pattern; PII detection (regex + NER + Luhn) with findings-not-values logging; prompt-injection defense (heuristic + LLM-secondary) scoped to a named corpus, reporting block *and* false-positive rates; atomic rate/spend limiting (INCR-and-check, race-aware); deterministic action allow-lists with HITL escalation; evaluating a *defense* (a different exercise from evaluating task success).

**Where these skills go next.** AI security/trust-and-safety is a growing, well-paid specialization. Interview questions: "How do you defend against prompt injection?", "How do you stop an agent from leaking PII?", "Why report false-positive rate alongside block rate?" Unlocks AI-security roles and any production-safety mandate. Resume phrasing: *"Built a reusable agent guardrail gateway (PII redaction ≥95% recall, injection block ≥85% at <5% false positives, atomic spend limits, action allow-list) protecting multiple agents by config only."*

**Builds on / feeds into.** Builds on: **01/02** (targets), **13** (reads the Contract's proposed actions). Feeds into: production-safety practice reused everywhere.

---

## 11 · Project 12 — Agent Release Gate (CI/CD for Agents)
*(ship agents like software)*

**What it is.** You package Project 03's eval harness as a reusable, installable **GitHub Action** that runs on every pull request, blocks the build on a regression, and posts a scorecard comment — demonstrated protecting *two* different repos by config only (no Action code changes). It's deliberately the least "agentic" project; the skill is packaging and reliability engineering — turning an eval harness into a devops product other teams install.

**Real-world use case.** This is what an **ML/AI platform or DevOps engineer** builds so an organization can practice eval-driven development — treating agent quality like test coverage, gated in CI. Business problem: without a gate, agent quality silently regresses on every prompt/model/code change; with one, regressions are caught before merge. Every team maturing from "we have an agent" to "we ship agents reliably" builds this.

**Skills learned.** CI/CD for non-deterministic systems (tolerance bands, not exact thresholds); packaging a prototype into a reusable, config-driven product; baseline-relative regression detection; GitHub Actions (composite/Docker actions, Checks API, the `pull_request` vs. `pull_request_target` fork-secret footgun); proving reusability by exercising it on a second repo.

**Where these skills go next.** "Eval-as-CI / agent release gating" is an emerging, valued practice. Interview questions: "How do you prevent agent quality regressions?", "How do you gate a non-deterministic system in CI without flaky failures?", "How do you make a CI tool reusable across repos?" Unlocks AI-platform/MLOps roles. Resume phrasing: *"Packaged an agent-eval harness as a reusable GitHub Action (baseline-relative regression gating, tolerance-banded to avoid flakiness); adopted by two repos via config only, blocking 3/3 regressed PRs."*

**Builds on / feeds into.** Builds on: **03** (the harness), **01 + 02** (target repos). Feeds into: the platform/MLOps skill set.

> ✅ **Resume Checkpoint 3 — production-grade.** With 02 + 11 + 12 you have the "I ship agents responsibly" story: durable HITL, reusable safety, and eval-as-CI.

---

## 12 · Project 16 — Multimodal Document Intelligence Agent
*(add vision — read charts, tables, scans)*

**What it is.** An agent that ingests layout-rich documents (PDFs with charts/tables/figures, scanned forms) and answers questions that require *reading the visuals*, citing the exact page and region. You build two pipelines — a text-only baseline (OCR + table-to-structured + figure captions) and a true visual pipeline (ColPali-style page-image retrieval) — and benchmark the vision *uplift* on chart/table questions. Citations are region-grounded: they resolve to a real `(page, bounding-box)`, the vision analogue of Project 06's file:line verification.

**Real-world use case.** This is what a **document-AI / applied-ML engineer** builds — visual document understanding is a huge market (invoice/receipt processing, financial-report analysis, insurance forms, medical records, contract review with figures). **In India**, this is **GST-invoice** parsing (GSTIN/HSN/tax-split tables), **KYC** card reading (masked Aadhaar/PAN under DPDP), and annual-report chart/table extraction — the document-AI behind Razorpay/PhonePe onboarding, lending/NBFC underwriting, and fintech expense products; India-built players like **Sarvam AI** and document-AI startups target exactly this. Business problem: real Indian business documents are visual — a text dump can't answer "what was the IGST on this invoice" or read a promoter-holding pie chart, and most RAG systems silently fail on exactly those questions.

**Skills learned.** Layout-aware document parsing (typed elements + bounding boxes); multimodal RAG (two strategies, benchmarked); vision-language generation over image regions; table extraction to structured cells (cell-level F1); visual retrieval (patch-level late interaction); region-grounded citation; measuring modality uplift against a baseline.

**Where these skills go next.** Multimodal is one of the strongest 2026 differentiators. Interview questions: "How do you do RAG over documents with charts and tables?", "OCR-then-RAG vs. visual retrieval — tradeoffs?", "How do you cite a visual answer?" Unlocks document-AI and multimodal-agent roles. Resume phrasing: *"Built a multimodal document-intelligence agent (layout parsing + visual retrieval) that outperformed a text-only baseline by 25+ points on chart/table questions, with region-grounded citations."*

**Builds on / feeds into.** Builds on: **10** (RAG), **01** (corpus), **06** (hybrid-fusion idea). Feeds into: multimodal breadth.

---

## 13 · Project 05 — Self-Healing SQL Analytics Agent
*(text-to-SQL with a benchmark number)*

**What it is.** An agent that turns a natural-language question into SQL, runs it against a read-only database, checks its own results for errors/emptiness/absurd values, and — within a bounded retry budget — fixes its own query before charting and narrating the answer. It's benchmarked on the Spider text-to-SQL benchmark so the resume claim is a *number*, not an adjective, and its safety (no destructive statements) is *structural* (read-only connection + a static SQL guard), not prompt-based.

**Real-world use case.** This is what a **data / analytics-engineering-adjacent AI engineer** builds — natural-language BI is a large market (every "ask your data a question" product: text-to-SQL copilots, analytics assistants, internal data-democratization tools). Business problem: analysts are a bottleneck for ad-hoc data questions, and NL-to-SQL removes it — but only if it's accurate and can't run a destructive query. Relevant across every data-heavy company.

**Skills learned.** Schema-grounded NL-to-SQL; bounded self-correction loops (feed the specific error back); cheap rule-based sanity checks before another LLM call; structural least-privilege safety (SQLite `mode=ro`/`PRAGMA` + a `sqlglot` statement guard); benchmark-driven evaluation with honest reporting.

**Where these skills go next.** Text-to-SQL is on many data/AI JDs. Interview questions: "How do you make NL-to-SQL safe?", "How does an agent self-correct a failed query?", "Why execution accuracy over string match?" Unlocks data-AI roles and is the port target for the local-models project. Resume phrasing: *"Built a self-healing text-to-SQL agent (bounded retry + structural read-only safety) benchmarked at 70%+ execution accuracy on Spider with a 60%+ self-heal rate."*

**Builds on / feeds into.** Builds on: nothing hard (self-contained). Feeds into: **18** (ported to local models — its structured-output-heavy nature is exactly where local models struggle).

---

## 14 · Project 18 — Local / Offline Open-Weight Agent
*(own the model — on-prem, private, offline)*

**What it is.** You re-implement Project 05 (recommended) to run *fully offline on local open-weight models* (Ollama/llama.cpp/vLLM), then benchmark it head-to-head against the hosted-API version on success, latency, and cost, and characterize the quality-vs-privacy-vs-cost frontier. The three hard problems — unreliable structured output on small models (solved with grammar-constrained decoding), small context + slow inference, and quantization quality loss (measured with a sweep) — are the real content.

**Real-world use case.** This is what an **AI platform / ML infrastructure engineer** builds for privacy-sensitive or cost-constrained deployments — healthcare, finance, defense, and any air-gapped or data-residency-bound environment where sending data to a hosted API is a non-starter, plus high-volume cases where self-hosting is cheaper. **In India this is a first-order concern**: the **DPDP Act** data-residency pressure means banks/hospitals/NBFCs often *cannot* send PII to a foreign API, ₹-cost at scale for price-sensitive users favors self-hosting, and low-connectivity regions need offline operation — which is exactly why India-built open models exist (**Sarvam**, **Krutrim**, **AI4Bharat**) to serve Hindi/Indic on-prem. Business problem: many Indian organizations *cannot* use hosted frontier APIs (compliance) or *shouldn't* at their volume (cost), and running agents on open weights is a genuinely different engineering discipline.

**Skills learned.** Self-hosting open-weight models (Ollama/llama.cpp/vLLM); quantization and the quality/latency/VRAM tradeoff (measured, not guessed); grammar-constrained / schema-guided decoding to make small-model structured output reliable by construction; offline RAG with local embeddings; producing a hosted-vs-local tradeoff analysis (the on-prem decision document); verifying "offline" by blocking egress.

**Where these skills go next.** On-prem/open-weight expertise is rare and well-paid (privacy-first startups, regulated enterprises, edge deployments). Interview questions: "When would you self-host vs. use an API?", "How do you make a small model reliable at tool calls?", "What does quantization trade off?" Unlocks ML-infra and privacy-sensitive AI roles. Resume phrasing: *"Ported an agent to fully-offline open-weight models (Ollama/llama.cpp) with grammar-constrained decoding (≥90% valid structured output) and characterized the hosted-vs-local quality/cost/privacy frontier across quantization levels."*

**Builds on / feeds into.** Builds on: **05** (the port target). Feeds into: ML-infra / on-prem specialization.

---

## 15 · Project 06 — Codebase Onboarding Agent
*(the live-on-their-repo demo)*

**What it is.** Point it at any GitHub repo: it indexes the code with AST-aware chunking (whole functions/classes, not fixed-size text), answers architecture questions with verified file:line citations, generates a README and a mermaid architecture diagram from a static import graph, and suggests scoped "first 5 issues" for new contributors. Its differentiators are AST-aware chunking (so citations are accurate) and combining static analysis (exact facts) with LLM synthesis (readable summaries).

**Real-world use case.** This is what a **developer-tools / platform-engineering AI engineer** builds — "chat with your codebase" and AI onboarding tools are a hot category (Cursor, Sourcegraph Cody, GitHub Copilot workspace-level features, and every internal "explain our monorepo" tool). Business problem: onboarding engineers to a large codebase is slow and expensive; an agent that answers architecture questions with trustworthy citations accelerates it. Relevant to any company with a big codebase.

**Skills learned.** AST-aware chunking (tree-sitter / `ast`); hybrid code retrieval (dense + BM25 for identifier matching); citation verification as a deterministic check; static analysis (import graphs) combined with LLM synthesis; handling oversized nodes; robustness on unpredictable live input.

**Where these skills go next.** Dev-tools is a large employer of agent engineers, and this is the most instantly demoable project (run it live on the interviewer's repo). Interview questions: "Why AST-aware chunking over fixed-size?", "How do you guarantee a citation is real?", "How do you build the architecture diagram accurately?" Unlocks dev-tools/platform roles. Resume phrasing: *"Built a codebase-onboarding agent (AST-aware chunking, hybrid code retrieval, 100% code-verified citations, static import-graph diagrams); runs live on any public repo."*

**Builds on / feeds into.** Builds on: RAG foundations (**01/10**). Feeds into: dev-tools specialization; its hybrid-retrieval idea feeds **16**.

---

## 16 · Project 15 — Computer-Use / Browser Automation Agent
*(act on a live GUI — the showstopper demo)*

**What it is.** Given a natural-language web task, it drives a real browser (Playwright) using the accessibility tree (primary) plus vision (fallback) to perceive the page, plans and executes actions (click/type/scroll), verifies progress, recovers from failures by re-planning, and gates any irreversible action (purchase/submit/delete) behind confirmation. Benchmarked on a fixed set of controlled web tasks for reproducible, safe scoring.

**Real-world use case.** This is what teams building **computer-use / RPA-next-generation agents** build (OpenAI Operator, Anthropic computer use, Adept-style agents, and the wave of "AI does tasks on the web for you" products). Business problem: enormous amounts of work happen through GUIs with no API — form-filling, data entry, cross-system workflows — and an agent that can operate a browser reliably automates them. One of the hottest 2026 capabilities and the strongest *live* demo you can show.

**Skills learned.** Perception grounding (accessibility-tree-first, vision fallback — the cost/reliability decision); typed action spaces; the plan/observe/act/verify loop; failure recovery and re-planning; structural safety gating for irreversible actions; benchmarking a fuzzy capability on controlled sites.

**Where these skills go next.** Computer-use is a marquee 2026 capability; a working live demo is rare and memorable. Interview questions: "DOM vs. vision grounding — when each?", "How do you make a browser agent safe?", "How does it recover when the page isn't what it expected?" Unlocks computer-use/automation roles. Resume phrasing: *"Built a computer-use browser agent (accessibility-tree + vision grounding, re-planning recovery, structural gate on irreversible actions) with 60%+ task success on a controlled web-task benchmark."*

**Builds on / feeds into.** Builds on: vision grounding (reuses ideas from **16**), safety gate (**04**). Feeds into: automation specialization.

---

## 17 · Project 17 — Voice / Real-Time Conversational Agent
*(the sub-second latency challenge)*

**What it is.** A real-time voice agent: the user speaks, it transcribes (streaming), reasons (reusing a portfolio agent as its brain), and speaks back — with natural turn-taking, interruption handling (barge-in), and a sub-second latency budget. The core engineering is the *latency budget* (turn detection + STT + brain + TTS, each sub-budgeted) and *pipelining* (start speaking sentence one while generating sentence two). It's the only project with a human-perceptible latency requirement.

**Real-world use case.** This is what teams building **voice AI** build — AI receptionists, phone support, drive-through ordering, scheduling agents, voice assistants (global players Vapi, Retell, Bland, LiveKit). **In India this is enormous and vernacular**: Hindi/Hinglish and multi-language phone support is the archetypal Indian voice-AI product — built by **Sarvam AI**, **Skit.ai (formerly Vernacular.ai)**, **Gnani.ai**, and used across banking IVR, collections, lending, and healthcare booking, on **AI4Bharat** open ASR/TTS models. Business problem: voice (in the user's own language) is the interface for huge swaths of Indian customer interaction — phone support and IVR at national scale — and a natural, interruptible, code-mixing-tolerant agent automates calls text bots can't. One of the fastest-growing 2026 categories, and especially so in India.

**Skills learned.** Real-time systems engineering (latency budgeting, streaming, pipelining vs. sequencing); streaming STT/LLM/TTS; turn detection (semantic, not just silence timers); barge-in / interruption handling (cancel in-flight TTS *and* brain); reusing a text agent as a modality-agnostic brain.

**Where these skills go next.** Voice AI is hiring aggressively. Interview questions: "How do you get a voice agent's latency under a second?", "How do you handle a user interrupting?", "How do you know when the user has finished speaking?" Unlocks voice-AI roles. Resume phrasing: *"Built a real-time voice agent (streaming STT/LLM/TTS, pipelined to P50 <1s end-to-end, barge-in <300ms, semantic turn detection) reusing an existing agent as its brain."*

**Builds on / feeds into.** Builds on: **04/01** (the brain), **13** (latency spans). Feeds into: voice specialization.

> ✅ **Resume Checkpoint 4 — multimodal breadth.** With 16 + 15 + 17 you cover every input modality (vision documents, GUI, voice) — very few candidates do.

---

## 18 · Project 07 — Red-Team vs Blue-Team Agent Arena
*(security specialization — defensive framing)*

**What it is.** Two agent crews in an isolated lab: an attacker crew probes a deliberately vulnerable app (OWASP Juice Shop) from a *fixed probe library* (the LLM only selects/sequences, never generates payloads) while a defender crew watches logs and proposes patches, refereed by a finite-state machine that enforces turn order in code. The deliverable is a *defensive-research* writeup — which vulnerability classes were found and how fast the defender detected and patched them (MTTD/MTTP). Everything is scoped, isolated, and framed around measurement and defense.

**Real-world use case.** This is what an **AI security engineer / SOC-automation engineer** explores — automated security testing and detection/response (companies doing AI-assisted red-teaming, SOC copilots, and continuous security validation). Business problem: security testing and incident detection are labor-intensive; agents can accelerate both, *if* built with strict scope and defensive framing. Relevant to security-adjacent AI roles. (Read the plan's scope/safety note first — this is deliberately defensive and lab-only.)

**Skills learned.** FSM-constrained multi-agent orchestration (turn order enforced in code, not prompts); isolated evaluation environments (verified network isolation); security metrics (coverage, MTTD, MTTP, distributions); defensive-research framing (the hiring-and-safety-appropriate way to present security work); a fixed-probe-library harness design.

**Where these skills go next.** Security-adjacent AI roles value demonstrated defensive maturity. Interview questions: "How do you enforce turn order in a multi-agent system structurally?", "How do you safely run an automated security exercise?", "Why measure MTTD/MTTP?" Unlocks AI-security roles. Resume phrasing: *"Built an isolated red-team/blue-team agent arena (FSM-enforced turns, fixed probe library, verified network isolation) measuring detection and patch-proposal latency against a defensive-research writeup."*

**Builds on / feeds into.** Builds on: orchestration (**01**), safety framing (portfolio-wide). Feeds into: security specialization.

---

## Skills-coverage matrix

Rows = projects (in build order). Columns = key 2026 skills. ●=primary focus, ○=meaningfully practiced. Read down a column to see how many angles you hit a skill from; read across a row to see a project's breadth.

| # | Project | Multi-agent orch. | Agentic RAG | Evals / LLM-judge | MCP | A2A | HITL | Memory | Deploy / Observability | Safety / Guardrails | Modality (vision/voice/local) |
|---|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1 | 01 Financial Research Analyst | ● | ● | ○ | | | ○ | | ● | | |
| 2 | 13 Agent Observability | ○ | | ○ | | | | | ● | | |
| 3 | 10 RAG Bake-Off | | ● | ○ | | | | | | | |
| 4 | 03 Eval & Observability Platform | ○ | | ● | | | | | ○ | ○ | |
| 5 | 09 MCP Server Trilogy | | | ○ | ● | | | | ○ | | |
| 6 | 04 MCP Personal Ops | ○ | | | ● | | ● | ○ | | ○ | |
| 7 | 14 Long-Term Memory | | ○ | ○ | | | | ● | | | |
| 8 | 08 A2A Multi-Framework | ● | | | ● | ● | | | ○ | | |
| 9 | 02 Document HITL Pipeline | ○ | | | | | ● | | ○ | ○ | |
| 10 | 11 Guardrail Gateway | | | ○ | | | ○ | | ○ | ● | |
| 11 | 12 Release Gate (CI/CD) | | | ● | | | | | ● | ○ | |
| 12 | 16 Multimodal Doc Intelligence | | ● | ○ | | | | | ○ | | ● |
| 13 | 05 Self-Healing SQL | ○ | | ○ | | | | | ○ | ● | |
| 14 | 18 Local / Offline | | ○ | ○ | | | | | ● | ○ | ● |
| 15 | 06 Codebase Onboarding | | ● | ○ | | | | | ○ | | |
| 16 | 15 Computer-Use / Browser | ○ | | ○ | | | ○ | | ○ | ● | ● |
| 17 | 17 Voice / Real-Time | ○ | | | | | | | ● | | ● |
| 18 | 07 Red-Team / Blue-Team | ● | | ○ | | | ○ | | ○ | ● | |
| | **Coverage (● count)** | **4** | **4** | **2** | **3** | **1** | **3** | **1** | **5** | **5** | **5** |

**How to read the coverage row.** Every 2026 skill is covered by at least one project where it's the *primary* focus, and most by several. The lightest single-● columns — **A2A** (08) and **Memory** (14) — are covered by exactly one deep project each *by design*: they're specialized protocols/systems, and one rigorous project that goes all the way (A2A with a real interop proof; memory with consolidation + a benchmark) is worth more than several shallow mentions. If you build only the **Minimum Viable** track (01 + 13 + 03), you get multi-agent orchestration, agentic RAG, evals, and deploy/observability — the four columns with the broadest market demand — which is why that trio is the recommended first shippable portfolio.
