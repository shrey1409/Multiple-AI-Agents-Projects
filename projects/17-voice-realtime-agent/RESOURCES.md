# RESOURCES.md — Voice / Real-Time Conversational Agent

Paths verified 2026-07-13 (HTTP 200). ✅ = confirmed.

## 1. What to clone / read

### Pipecat — real-time voice-agent orchestration framework (primary)
```bash
git clone --depth 1 https://github.com/pipecat-ai/pipecat.git /tmp/ref-pipecat   # ✅
```
Handles the real-time pipeline (VAD → streaming STT → LLM → streaming TTS), interruption, and transport. Build your loop on this rather than hand-rolling WebRTC. Read its pipeline/interruption examples — they map directly to PLAN §2.

### LiveKit Agents — alternative voice-agent framework (read/choose)
```bash
git clone --depth 1 https://github.com/livekit/agents.git /tmp/ref-livekit-agents   # ✅
```
A strong alternative with built-in turn detection and telephony. Pick Pipecat or LiveKit; both cover the same loop — choose by transport needs (LiveKit shines for phone/WebRTC at scale).

### Whisper — STT reference (the 500-repo voice entry)
```bash
git clone --depth 1 https://github.com/openai/whisper.git /tmp/ref-whisper   # ✅
```
The 500-repo's "Agent Chat with Whisper" entry. For *streaming* STT use a streaming ASR/service (batch Whisper adds full-utterance latency) — Whisper here is the reference for the transcription concept and for offline/local STT if you go that route.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| Pipecat | `pipecat-ai/pipecat` | Reuse — real-time pipeline + interruption | 0–4 |
| LiveKit Agents | `livekit/agents` | Alternative — choose one | 0–4 |
| Whisper | `openai/whisper` | Read-only / reuse — STT concept + local STT option | 0 |

## 3. The brain (internal reuse)
Reuse Project 04 (personal ops — the most natural voice-assistant demo: calendar/PRs/notes by voice) or Project 01 (research by voice) as the brain over its Target Agent Contract API. Do not build a new reasoning agent.

## 4. Data
A scripted call set: 15 spoken tasks with reference transcripts + expected outcomes, for reproducible latency/WER/task-success measurement. Instrument each pipeline stage with Project 13's OTel spans to measure the latency budget.

## 4. India-specific language stack (localization)
Real-time pipeline unchanged; swap in Indian-language models:
- **STT:** OpenAI **Whisper** (`openai/whisper`, verified) as default (Indian-accented English + Hindi). Higher accuracy for Indic: **AI4Bharat** ASR (IndicConformer / IndicWav2Vec) — `github.com/AI4Bharat` (org verified; models on HuggingFace `ai4bharat/*`).
- **TTS:** **AI4Bharat Indic-TTS** (`github.com/AI4Bharat/Indic-TTS`, verified) for natural Indian voices; or a hosted Indian-voice TTS API.
- **Hinglish / code-mixing:** `indic-transliteration` (PyPI, verified) + IndicTrans2 (`AI4Bharat/IndicTrans2`, verified) for script/lang normalization.
- **India-built LLM options for the brain** (optional): Sarvam, Krutrim families (Indian LLM startups; check current HF/API availability at build time).
- **Use case:** vernacular phone-support / appointment-booking bot; the brain is Project 04 or 01 (unchanged) behind the voice layer.
