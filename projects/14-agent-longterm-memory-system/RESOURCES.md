# RESOURCES.md — Agent Long-Term Memory System

Paths verified 2026-07-13 (HTTP 200). ✅ = confirmed.

## 1. What to clone / read

### mem0 — memory-layer design reference (read-only, primary)
```bash
git clone --depth 1 https://github.com/mem0ai/mem0.git /tmp/ref-mem0   # ✅
```
The closest prior art: a memory layer with add/search/update and fact extraction. Read how it does extraction, dedup, and update (its "add" pipeline decides insert vs. update vs. no-op) — then build your own typed/consolidating version. Don't adopt wholesale; the point is to build the policies yourself.

### Letta (formerly MemGPT) — memory-management design reference (read-only)
```bash
git clone --depth 1 https://github.com/letta-ai/letta.git /tmp/ref-letta   # ✅
```
Read for the core/archival memory split and self-editing memory ideas — a different architecture than yours, useful contrast for your episodic/semantic/procedural model.

### getzep/zep — temporal memory + fact invalidation reference (read-only)
```bash
git clone --depth 1 https://github.com/getzep/zep.git /tmp/ref-zep   # ✅
```
Read specifically for how it handles *temporal* facts and invalidation (its take on supersession) — directly relevant to your contradiction-handling policy.

### pgvector (your store)
```bash
git clone --depth 1 https://github.com/pgvector/pgvector.git /tmp/ref-pgvector   # ✅ master
pip install pgvector   # + the Postgres extension
```
The vector column type + similarity operators for your Postgres-backed store (episodic/semantic/procedural + metadata queries in one DB).

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| mem0 | repo | Read-only — extraction/dedup/update pipeline | 1, 3 |
| Letta | repo | Read-only — memory-management architecture contrast | 0, 2 |
| Zep | repo | Read-only — temporal facts + invalidation (supersession) | 4 |
| pgvector | repo (master) | Reuse — vector store | 0 |

## 3. Data (synthesize, don't source)
Generate a multi-session conversation dataset with ground-truth recall Q&A (PLAN §5): ~10 multi-turn sessions per persona, later sessions testing recall of earlier facts, plus explicit *contradiction* arcs (preference changes) and *consolidation* arcs (a preference stated implicitly across 3 sessions). No real user data.

## 4. Reused across the portfolio
- Project 04 is the first consumer — its inline memory gets swapped for this service (the reuse proof).
- Project 01's embedding choice/model is reused for consistency.
