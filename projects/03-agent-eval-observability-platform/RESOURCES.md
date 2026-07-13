# RESOURCES.md — Agent Evaluation & Observability Platform

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed (HTTP 200 or present in clone).

## 1. What to clone / download

### LangGraph chatbot-simulation-evaluation tutorial
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Moved to `examples/` — verified path: **`examples/chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb`** ✅ (there's also a `langsmith-agent-simulation-evaluation.ipynb` variant). The single most directly relevant reference — adapt its simulator↔target loop; you extend it with adversarial personas + a separate judge pass.

### AutoGen AgentEval + AgentOps notebooks (read-only)
```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agenteval_cq_math.ipynb        # ✅ 200
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_agentops.ipynb        # ✅ 200
```
⚠️ These are on the **`0.2` branch**, which is **legacy** — the AutoGen repo is in maintenance mode (superseded by `autogen-agentchat` 0.4+ / Microsoft Agent Framework). Fine as read-only design references: AgentEval shows a criteria-generation-then-quantify "panel of judges" alternative; AgentOps shows what to instrument (tokens, tool errors, latency) — cross-check your Trace/Judgment schema against it.

### RAGAS (if the target does RAG)
```bash
git clone --depth 1 https://github.com/explodinggradients/ragas.git /tmp/ref-ragas   # ✅
pip install ragas
```
Gives faithfulness/answer-relevancy/context-precision for free if the target (Project 01) exposes retrieved context in its trajectory. Check current metric import names against the installed version.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| LangGraph simulation-eval | `examples/chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb` | Adapt — simulator↔target loop | 1, 2 |
| AutoGen AgentEval (0.2, legacy) | `notebook/agenteval_cq_math.ipynb` | Read-only — panel-of-judges alternative | 3 |
| AutoGen AgentOps (0.2, legacy) | `notebook/agentchat_agentops.ipynb` | Read-only — instrumentation fields | 0, 1 |
| RAGAS | repo/`pip install ragas` | Reuse — RAG-subcomponent metrics | 3 |

## 3. Also load-bearing (non-code)
- **OWASP LLM Top 10** (`github.com/OWASP/www-project-top-10-for-large-language-model-applications` ✅) — read LLM01 (prompt injection) before writing the adversarial personas so they reflect real patterns.
- **Cross-project:** the Target Agent Contract this harness consumes is defined in Projects 01/02 §2 and implemented as OTel export by Project 13. Read those before Phase 0.
