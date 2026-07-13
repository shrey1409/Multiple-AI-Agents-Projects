# RESOURCES.md — Local / Offline Open-Weight Agent

Paths verified 2026-07-13 (HTTP 200). ✅ = confirmed.

## 1. What to clone / read

### Ollama — easiest local model serving (dev)
```bash
git clone --depth 1 https://github.com/ollama/ollama.git /tmp/ref-ollama   # ✅
# Then: ollama pull qwen2.5:7b-instruct  (or a similar open-weight instruct model)
```
Simplest way to serve a quantized open-weight model locally with an OpenAI-compatible endpoint — swap your agent's base URL and you're running local. Use for the build.

### llama.cpp — grammar-constrained decoding (the reliability mitigation)
```bash
git clone --depth 1 https://github.com/ggml-org/llama.cpp.git /tmp/ref-llamacpp   # ✅ (ggml-org, master)
```
Read the **GBNF grammar** docs/examples — this is how you force valid JSON/tool-call output by construction on a small model (PLAN §2, Phase 2). Also the reference for GGUF quantization levels (Q4_K_M, Q8_0, …) for the sweep.

### vLLM — high-throughput serving (perf/scale)
```bash
git clone --depth 1 https://github.com/vllm-project/vllm.git /tmp/ref-vllm   # ✅
```
For the throughput/cost part of the analysis (batched serving) if you push past single-request Ollama. Read for how production self-hosting scales.

### LangGraph "Local" RAG notebooks (the 500-repo entries that motivated this project)
```bash
git clone --depth 1 https://github.com/langchain-ai/langgraph.git /tmp/ref-langgraph
```
Verified present (moved to `examples/`): `examples/rag/langgraph_adaptive_rag_local.ipynb` ✅ and `examples/rag/langgraph_self_rag_local.ipynb` ✅ — the "Adaptive RAG (Local)" / "Self-RAG (Local)" rows. Read for how they wire local models + local embeddings into a RAG graph offline.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| Ollama | `ollama/ollama` | Reuse — local serving (dev) | 0, 1 |
| llama.cpp GBNF | `ggml-org/llama.cpp` | Reuse — grammar-constrained JSON + quant levels | 2, 4 |
| vLLM | `vllm-project/vllm` | Read-only / reuse — throughput serving | 4 |
| LangGraph local RAG | `examples/rag/langgraph_{adaptive_rag,self_rag}_local.ipynb` | Adapt — offline RAG wiring | 3 |

## 3. The agent to port (internal reuse)
Port **Project 05** (self-healing SQL — structured-output-heavy, so the grammar-constraint mitigation shines) or **Project 01's fundamentals RAG** (for the offline-RAG angle). Reuse the ported project's **exact eval set** so the hosted-vs-local comparison is clean. Keep the hosted version wired as the baseline.

## 4. Also useful
- `outlines` (JSON-schema-guided decoding) as an alternative to GBNF if you serve via vLLM.
- A local embedding model (e.g., a small `bge`/`gte` GGUF or sentence-transformers offline) so RAG is genuinely offline.

## 4. India-specific motivation + models (localization)
Systems curriculum unchanged; India-tuned drivers and models:
- **Runtimes** (unchanged): Ollama (`ollama/ollama`), llama.cpp (`ggml-org/llama.cpp`), vLLM (`vllm-project/vllm`) — all verified.
- **Indian-language open models:** **AI4Bharat** models (`github.com/AI4Bharat`, verified) for Indic; India-built LLMs (Sarvam, Krutrim families) for on-prem Hindi/Indic serving — pairs with Project 17 (offline vernacular voice) and Project 02/11 (on-prem PII under DPDP).
- **Why local, in India:** **DPDP Act** data-residency (a bank/hospital may not send PII to a foreign API), **₹ cost** at scale, and **low-connectivity** offline operation. Report the hosted-vs-local quality/cost gap in ₹ — the honest measurement is the deliverable.
